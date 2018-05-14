# Get Started with TensorBoard

TensorBoard is a suite of web applications for inspecting and understanding your
TensorFlow runs and graphs.

This README gives an overview of key concepts in TensorBoard, as well as how to
interpret the visualizations TensorBoard provides. For an in-depth example of
using TensorBoard, see the tutorial: [TensorBoard: Visualizing
Learning][].
For in-depth information on the Graph Visualizer, see this tutorial: 
[TensorBoard: Graph Visualization][].

[TensorBoard: Visualizing Learning]: https://www.tensorflow.org/get_started/summaries_and_tensorboard
[TensorBoard: Graph Visualization]: https://www.tensorflow.org/get_started/graph_viz

You may also want to watch
[this video tutorial][] that walks
through setting up and using TensorBoard. There's an associated 
[tutorial with an end-to-end example of training TensorFlow and using TensorBoard][].

[this video tutorial]: https://www.youtube.com/watch?v=eBbEDRsCmv4

[tutorial with an end-to-end example of training TensorFlow and using TensorBoard]: https://github.com/dandelionmane/tf-dev-summit-tensorboard-tutorial

# Usage

Before running TensorBoard, make sure you have generated summary data in a log
directory by creating a summary writer:

``` python
# sess.graph contains the graph definition; that enables the Graph Visualizer.

file_writer = tf.summary.FileWriter('/path/to/logs', sess.graph)
```

For more details, see 
[the TensorBoard tutorial](https://www.tensorflow.org/get_started/summaries_and_tensorboard).
Once you have event files, run TensorBoard and provide the log directory. If
you're using a precompiled TensorFlow package (e.g. you installed via pip), run:

```
tensorboard --logdir path/to/logs
```

Or, if you are building from source:

```bash
bazel build tensorboard:tensorboard
./bazel-bin/tensorboard/tensorboard --logdir path/to/logs

# or even more succinctly
bazel run tensorboard -- --logdir path/to/logs
```

This should print that TensorBoard has started. Next, connect to
http://localhost:6006.

TensorBoard requires a `logdir` to read logs from. For info on configuring
TensorBoard, run `tensorboard --help`.

TensorBoard can be used in Google Chrome or Firefox. Other browsers might
work, but there may be bugs or performance issues.

# Key Concepts

### Summary Ops: How TensorBoard gets data from TensorFlow

The first step in using TensorBoard is acquiring data from your TensorFlow run.
For this, you need 
[summary ops](https://www.tensorflow.org/api_docs/python/tf/summary).
Summary ops are ops, like
[`tf.matmul`](https://www.tensorflow.org/versions/r1.2/api_docs/python/tf/matmul)
or
[`tf.nn.relu`](https://www.tensorflow.org/versions/master/api_docs/python/tf/nn/relu),
which means they take in tensors, produce tensors, and are evaluated from within
a TensorFlow graph. However, summary ops have a twist: the Tensors they produce
contain serialized protobufs, which are written to disk and sent to TensorBoard.
To visualize the summary data in TensorBoard, you should evaluate the summary
op, retrieve the result, and then write that result to disk using a
summary.FileWriter. A full explanation, with examples, is in [the
tutorial](https://www.tensorflow.org/get_started/summaries_and_tensorboard).

The supported summary ops include:
* [`tf.summary.scalar`](https://www.tensorflow.org/api_docs/python/tf/summary/scalar)
* [`tf.summary.image`](https://www.tensorflow.org/api_docs/python/tf/summary/image)
* [`tf.summary.audio`](https://www.tensorflow.org/api_docs/python/tf/summary/audio)
* [`tf.summary.text`](https://www.tensorflow.org/api_docs/python/tf/summary/text)
* [`tf.summary.histogram`](https://www.tensorflow.org/api_docs/python/tf/summary/histogram)

### Tags: Giving names to data

When you make a summary op, you will also give it a `tag`. The tag is basically
a name for the data recorded by that op, and will be used to organize the data
in the frontend. The scalar and histogram dashboards organize data by tag, and
group the tags into folders according to a directory/like/hierarchy. If you have
a lot of tags, we recommend grouping them with slashes.

### Event Files & LogDirs: How TensorBoard loads the data

`summary.FileWriters` take summary data from TensorFlow, and then write them to a
specified directory, known as the `logdir`. Specifically, the data is written to
an append-only record dump that will have "tfevents" in the filename.
TensorBoard reads data from a full directory, and organizes it into the history
of a single TensorFlow execution.

Why does it read the whole directory, rather than an individual file? You might
have been using
[supervisor.py](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/python/training/supervisor.py)
to run your model, in which case if TensorFlow crashes, the supervisor will
restart it from a checkpoint. When it restarts, it will start writing to a new
events file, and TensorBoard will stitch the various event files together to
produce a consistent history of what happened.

### Runs: Comparing different executions of your model

You may want to visually compare multiple executions of your model; for example,
suppose you've changed the hyperparameters and want to see if it's converging
faster. TensorBoard enables this through different "runs". When TensorBoard is
passed a `logdir` at startup, it recursively walks the directory tree rooted at
`logdir` looking for subdirectories that contain tfevents data. Every time it
encounters such a subdirectory, it loads it as a new `run`, and the frontend
will organize the data accordingly.

For example, here is a well-organized TensorBoard log directory, with two runs,
"run1" and "run2".

```
/some/path/mnist_experiments/
/some/path/mnist_experiments/run1/
/some/path/mnist_experiments/run1/events.out.tfevents.1456525581.name
/some/path/mnist_experiments/run1/events.out.tfevents.1456525585.name
/some/path/mnist_experiments/run2/
/some/path/mnist_experiments/run2/events.out.tfevents.1456525385.name
/tensorboard --logdir /some/path/mnist_experiments
```

You may also pass a comma separated list of log directories, and TensorBoard
will watch each directory. You can also assign names to individual log
directories by putting a colon between the name and the path, as in

```
tensorboard --logdir name1:/path/to/logs/1,name2:/path/to/logs/2
```

# The Visualizations

### Scalar Dashboard

TensorBoard's Scalar Dashboard visualizes scalar statistics that vary over time;
for example, you might want to track the model's loss or learning rate. As
described in *Key Concepts*, you can compare multiple runs, and the data is
organized by tag. The line charts have the following interactions:

* Clicking on the small blue icon in the lower-left corner of each chart will
expand the chart

* Dragging a rectangular region on the chart will zoom in

* Double clicking on the chart will zoom out

* Mousing over the chart will produce crosshairs, with data values recorded in
the run-selector on the left.

Additionally, you can create new folders to organize tags by writing regular
expressions in the box in the top-left of the dashboard.

### Histogram Dashboard

The HistogramDashboard displays how the statistical distribution of a Tensor
has varied over time. It visualizes data recorded via `tf.summary.histogram`.
Each chart shows temporal "slices" of data, where each slice is a histogram of
the tensor at a given step. It's organized with the oldest timestep in the back,
and the most recent timestep in front. By changing the Histogram Mode from
"offset" to "overlay", the perspective will rotate so that every histogram slice
is rendered as a line and overlaid with one another.

### Distribution Dashboard

The Distribution Dashboard is another way of visualizing histogram data from
`tf.summary.histogram`. It shows some high-level statistics on a distribution.
Each line on the chart represents a percentile in the distribution over the
data: for example, the bottom line shows how the minimum value has changed over
time, and the line in the middle shows how the median has changed. Reading from
top to bottom, the lines have the following meaning: `[maximum, 93%, 84%, 69%,
50%, 31%, 16%, 7%, minimum]`

These percentiles can also be viewed as standard deviation boundaries on a
normal distribution: `[maximum, μ+1.5σ, μ+σ, μ+0.5σ, μ, μ-0.5σ, μ-σ, μ-1.5σ,
minimum]` so that the colored regions, read from inside to outside, have widths
`[σ, 2σ, 3σ]` respectively.


### Image Dashboard

The Image Dashboard can display pngs that were saved via a `tf.summary.image`.
The dashboard is set up so that each row corresponds to a different tag, and
each column corresponds to a run. Since the image dashboard supports arbitrary
pngs, you can use this to embed custom visualizations (e.g. matplotlib
scatterplots) into TensorBoard. This dashboard always shows you the latest image
for each tag.

### Audio Dashboard

The Audio Dashboard can embed playable audio widgets for audio saved via a
`tf.summary.audio`. The dashboard is set up so that each row corresponds to a
different tag, and each column corresponds to a run. This dashboard always
embeds the latest audio for each tag.

### Graph Explorer

The Graph Explorer can visualize a TensorBoard graph, enabling inspection of the
TensorFlow model. To get best use of the graph visualizer, you should use name
scopes to hierarchically group the ops in your graph - otherwise, the graph may
be difficult to decipher. For more information, including examples, see [the
graph visualizer tutorial](https://www.tensorflow.org/get_started/graph_viz).

### Embedding Projector

The Embedding Projector allows you to visualize high-dimensional data; for
example, you may view your input data after it has been embedded in a high-
dimensional space by your model. The embedding projector reads data from your
model checkpoint file, and may be configured with additional metadata, like
a vocabulary file or sprite images. For more details, see [the embedding
projector tutorial](https://www.tensorflow.org/get_started/embedding_viz).

### Text Dashboard

The Text Dashboard displays text snippets saved via `tf.summary.text`. Markdown
features including hyperlinks, lists, and tables are all supported.