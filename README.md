# Inception v3 Android Camera Demo

This folder contains an example application utilizing TensorFlow for Android devices.

[how does it works](https://youtu.be/na2CQGZndNc)

# Dependencies:
- Python 3.6
- Android Studio 3.1.3
- Android SDK Tools 26.1.1
- Tensorflow 1.7 (TOCO will not work with 1.8 so be sure that you install Tensorflow 1.70

## What you should do with your trained Inception v3 model before you run it on Android?
Firstly, after successful training(in our case - transfer-learning) your model, you will get at least two files, first is output_labels.txt(it contain class names), and second is output_graph.pb(it is your trained model). If you use Iception v3 model you should strip the DecodeJpeg layer from your retrained model first, you can easily do this by running this script:
```shell
python strip_unused.py \
--input_graph=<retrained-pb-file> \
--output_graph=<your-stripped-pb-file> \
--input_node_names="Mul" \
--output_node_names="final_result" \
--input_binary=true
```
The next step is to convert your model(*.pb) into a special format. To do this you need to consistently run the scripts shown below: 

### Optimize your graph: 
```shell
bazel build tensorflow/python/tools:optimize_for_inference
bazel-bin/tensorflow/python/tools/optimize_for_inference \
--input=<stripped-pb-file> \
--output=<your-optimized-pb-file> \
--input_names=Mul \
--output_names=final_result
```
### Quantize your graph:
```shell
bazel build tensorflow/tools/quantization:quantize_graph
bazel-bin/tensorflow/tools/quantization/quantize_graph \
--input=<optimized-pb-file> \
--output=<your-rounded-pb-file> \
--output_node_names=final_result \
--mode=weights_rounded
```
At result you will have five files like this output_labels.txt output_graph.pb, striped_graph.pb, optimized_graph.pb and rounded_graph.pb. For deploying on Android you need just the last file(rounded_graph.pb) and file which contain class names(output_labels.txt)

## Okey, you have rounded-pb-file(model) and output-txt-file(class names) what is the next?
You just need move these files into android/assets/, theirs names should be rounded_graph.pb and output_labels.txt.Also you can use another names for these files you just need edit this in android/src/org/tensorflow/demo/ClassifierActivity.java on rows 79 and 80.
```java
private static final String MODEL_FILE = "file:///android_asset/rounded_graph.pb";
private static final String LABEL_FILE = "file:///android_asset/output_labels.txt";
```
## The last step is to build your project, install an apk for your device and try this app on some products from output_label.txt.