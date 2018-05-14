# Frequently Asked Questions about TensorBoard

### My TensorBoard isn't showing any data! What's wrong?

First, check that the directory passed to `--logdir` is correct. You can also
verify this by navigating to the Scalars dashboard (under the "Inactive" menu)
and looking for the log directory path at the bottom of the left sidebar.

If you're loading from the proper path, make sure that event files are present.
TensorBoard will recursively walk its logdir, it's fine if the data is nested
under a subdirectory. Ensure the following shows at least one result:

`find DIRECTORY_PATH | grep tfevents`

You can also check that the event files actually have data by running
tensorboard in inspect mode to inspect the contents of your event files.

`tensorboard --inspect --logdir DIRECTORY_PATH`

### TensorBoard is showing only some of my data, or isn't properly updating!

This issue usually comes about because of how TensorBoard iterates through the
`tfevents` files: it progresses through the events file in timestamp order, and
only reads one file at a time. Let's suppose we have files with timestamps `a`
and `b`, where `a<b`. Once TensorBoard has read all the events in `a`, it will
never return to it, because it assumes any new events are being written in the
more recent file. This could cause an issue if, for example, you have two
`FileWriters` simultaneously writing to the same directory. If you have
multiple summary writers, each one should be writing to a separate directory.

### Does TensorBoard support multiple or distributed summary writers?

No. TensorBoard expects that only one events file will be written to at a time,
and multiple summary writers means multiple events files. If you are running a
distributed TensorFlow instance, we encourage you to designate a single worker
as the "chief" that is responsible for all summary processing. See
[supervisor.py](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/python/training/supervisor.py)
for an example.

### I'm seeing data overlapped on itself! What gives?

If you are seeing data that seems to travel backwards through time and overlap
with itself, there are a few possible explanations.

* You may have multiple execution of TensorFlow that all wrote to the same log
directory. Please have each TensorFlow run write to its own logdir.

* You may have a bug in your code where the global_step variable (passed
to `FileWriter.add_summary`) is being maintained incorrectly.

* It may be that your TensorFlow job crashed, and was restarted from an earlier
checkpoint. See *How to handle TensorFlow restarts*, below.

As a workaround, try changing the x-axis display in TensorBoard from `steps` to
`wall_time`. This will frequently clear up the issue.

### How should I handle TensorFlow restarts?

TensorFlow is designed with a mechanism for graceful recovery if a job crashes
or is killed: TensorFlow can periodically write model checkpoint files, which
enable you to restart TensorFlow without losing all your training progress.

However, this can complicate things for TensorBoard; imagine that TensorFlow
wrote a checkpoint at step `a`, and then continued running until step `b`, and
then crashed and restarted at timestamp `a`. All of the events written between
`a` and `b` were "orphaned" by the restart event and should be removed.

To facilitate this, we have a `SessionLog` message in
`tensorflow/core/util/event.proto` which can record `SessionStatus.START` as an
event; like all events, it may have a `step` associated with it. If TensorBoard
detects a `SessionStatus.START` event with step `a`, it will assume that every
event with a step greater than `a` was orphaned, and it will discard those
events. This behavior may be disabled with the flag
`--purge_orphaned_data false` (in versions after 0.7).

### How can I export data from TensorBoard?

The Scalar Dashboard supports exporting data; you can click the "enable
download links" option in the left-hand bar. Then, each plot will provide
download links for the data it contains.

If you need access to the full dataset, you can read the event files that
TensorBoard consumes by using the [`summary_iterator`](
https://www.tensorflow.org/api_docs/python/tf/train/summary_iterator)
method.

### Can I customize which lines appear in a plot?

Using the [custom scalars plugin](tensorboard/plugins/custom_scalar), you can
create scalar plots with lines for custom run-tag pairs. However, within the
original scalars dashboard, each scalar plot corresponds to data for a specific
tag and contains lines for each run that includes that tag.

### Can I visualize margins above and below lines?

Margin plots (that visualize lower and upper bounds) may be created with the
[custom scalars plugin](tensorboard/plugins/custom_scalar). The original
scalars plugin does not support visualizing margins.

### Can I create scatterplots (or other custom plots)?

This isn't yet possible. As a workaround, you could create your custom plot in
your own code (e.g. matplotlib) and then write it into an `SummaryProto`
(`core/framework/summary.proto`) and add it to your `FileWriter`. Then, your
custom plot will appear in the TensorBoard image tab.

### Is my data being downsampled? Am I really seeing all the data?

TensorBoard uses [reservoir
sampling](https://en.wikipedia.org/wiki/Reservoir_sampling) to downsample your
data so that it can be loaded into RAM. You can modify the number of elements it
will keep per tag in
[tensorboard/backend/application.py](tensorboard/backend/application.py).
See this [StackOverflow question](http://stackoverflow.com/questions/43702546/tensorboard-doesnt-show-all-data-points/)
for some more information.

### I get a network security popup every time I run TensorBoard on a mac!

This is because by default, TensorBoard serves on host `0.0.0.0` which is
publicly accessible. You can stop the popups by specifying `--host localhost` at
startup.

### How can I contribute to TensorBoard development?

See [DEVELOPMENT.md](DEVELOPMENT.md).

### I have a different issue that wasn't addressed here!

First, try searching our [GitHub
issues](https://github.com/tensorflow/tensorboard/issues) and [Stack
Overflow][stack-overflow]. It may be
that someone else has already had the same issue or question.

General usage questions (or problems that may be specific to your local setup)
should go to [Stack Overflow][stack-overflow].

If you have found a bug in TensorBoard, please [file a GitHub issue](
https://github.com/tensorflow/tensorboard/issues/new) with as much supporting
information as you can provide (e.g. attaching events files, including the output
of `tensorboard --inspect`, etc.).

[stack-overflow]: https://stackoverflow.com/questions/tagged/tensorboard