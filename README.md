# video-rants

# Nvenc 608/708 close captions:

Explain this line:
https://gitlab.freedesktop.org/gstreamer/gstreamer/-/blob/main/subprojects/gst-plugins-bad/sys/nvcodec/gstnvencoder.cpp#L1872

How tu put here EIA608 two byte captions?
https://gitlab.freedesktop.org/gstreamer/gstreamer/-/blob/main/subprojects/gst-plugins-bad/sys/nvcodec/gstnvencoder.cpp#L1880

The first byte of each triplet should look like that:
// padding     | valid | NTSC_FIELD0 triplet for raw eia608 type
// 1 1 1 1 1   | 1     | 0 0

--

# timestamp weirdness

static uint64 get_brt_from_wct_ns (AmgCapsequoSubtitleReceiver *filter, guint64 wct_ns)
{
    int64 wct_map_ns = 0, rt_map_ns = 0, ret_brt_ns = 0;
    get_wct_rt_map_ns(filter, &wct_map_ns, &rt_map_ns);

	// wct_ns - timestamp from capsequo/alta captioning services (epoch, ns tb)
	// wct_map_ns - wallclock timestamp at the receiving instance (epoch, ns tb)
	// rt_map_ns - gstreamre pipeline clock timestamp in ns (starting from 0, ns tb)
	// the result of this arithmetic was a huge epoch timestamp number during the first few seconds after pipeline start
	// why? how could we fix that?
    return rt_map_ns + (wct_ns - wct_map_ns);
}

--

# fast replays

https://docs.obsproject.com/reference-libobs-util-circlebuf

static struct obs_source_frame* replay_filter_video(void *data, struct obs_source_frame *frame) {
    struct replay_filter *filter = data;

    struct obs_source_frame *new_frame = obs_source_frame_create(frame->format, frame->width, frame->height);
    new_frame->refs = 1;
    obs_source_frame_copy(new_frame, frame);
    timestamp_stabilizer(filter, new_frame);

    pthread_mutex_lock(&filter->mutex);
    circlebuf_push_back(&filter->video_frames, &new_frame, sizeofOBS replays plugin:
}

struct replay {
	// TODO: what would be the type here? video_frames;
    // (...)
};

void replay_retrieve(struct replay_source *context)
{
    obs_source_t *s = obs_get_source_by_name(context->source_name);
    struct replay_filter *vf = context->source_filter;

    struct replay new_replay;

    pthread_mutex_lock(&vf->mutex);

    struct obs_source_frame *frame;

    for (int i = 0; i < new_replay.video_frame_count; i++) {
        // fetch the pointers to frame pointers from the circlebuf

        // TODO: fill this part with code
        // the retrieve method needs to be fast, in terms of 10s of millis - the user can spam the grab replay button with HW devices
    }

    pthread_mutex_unlock(&vf->mutex);

--

# C++ mse related:

  https://github.com/amagimedia/avplumber

  https://github.com/amagimedia/avplumber/tree/develop/doc

  Explain "Object" token in the context of InstanceSharedObjects class
  https://github.com/amagimedia/avplumber/blob/cc_passthru/src/instance_shared.hpp

