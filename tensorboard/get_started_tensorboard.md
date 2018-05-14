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

