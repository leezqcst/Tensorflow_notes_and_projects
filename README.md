# Tensorflow projects
- `tf_1_linear_regression.ipynb`: linear and polynomial regression.
- `tf_2_word2vec.ipynb`: word2vec. See [Chinese notes](http://url.cn/5PKmy7W), [中文解读](http://url.cn/5PKmy7W).
- `tf_3_LSTM_text_classification_version_1.ipynb`: LSTM for text classification version 1. `tf.nn.static_rnn` with single layer. See [Chinese notes](http://url.cn/5cLDOQI), [中文解读](http://url.cn/5cLDOQI).
- `tf_3_LSTM_text_classification_version_2.ipynb`: LSTM for text classification version 2. `tf.nn.dynamic_rnn` with multiple layers, variable sequence length, last relevant output. See [Chinese notes](http://url.cn/5w5VbaI), [中文解读](http://url.cn/5w5VbaI).
- `tf_4_bi-directional_LSTM_NER.ipynb`: bi-directional LSTM + CRF for brands NER. See [English notes](https://github.com/gaoisbest/NLP-Projects/blob/master/Sequence%20labeling%20-%20NER/README.md), [Chinese notes](http://url.cn/5fcC754) and [中文解读](http://url.cn/5fcC754).
- `tf_5_CNN_text_classification.ipynb`: CNN for text classification. `tf.nn.conv2d`, `tf.nn.max_pool`. See [Chinese notes](http://url.cn/5kW61T4), [中文解读](http://url.cn/5kW61T4).

# Tensorflow notes
`Lectures 1-2.md`, `Lectures 3.md` and `Lectures 4-5.md` are notes of [cs20si](http://web.stanford.edu/class/cs20si/). Each lecture includes basic concepts, codes and part solutions of corresponding assignment.

# RNNs
## 1. Difference between `tf.nn.static_rnn` and `tf.nn.dynamic_rnn`.
- `static_rnn` creates an **unrolled** RNNs network by chaining cells. The weights are shared between cells. Since the network is static, the input length should be same.
- `dynamic_rnn` uses a `while_loop()`operation to run over the cell the appropriate number of times.
- Both have `sequence_length` parameter, which is a `batch_size` 1D tensor . When exceed `sequence_length`, they will **copy-through state and zero-out outputs**.

References:  
[1] Hands on machine learning with Scikit-Learn and TensorFlow p385  
[2] https://www.zhihu.com/question/52200883

## 2. How to perform series output at RNNs each time step ?
Under the scenarios of stock price prediction and char-rnn etc, we want to obtain the outputs of each time step. Basically, there are two methods, but the first is not efficient (many fully connected layers for all time steps). So the second is perfered (only one fully connected layer).
- `tf.contrib.rnn.OutputProjectionWrapper` **adds a fully connected layer without activation** on top of each time step output, but not affect the cell state. And all the fully connected layers share the same weights and biases. The *projection* means linear transformation without activation.
  - Usage: `cell = tf.contrib.rnn.OutputProjectionWrapper(cell, output_size = 1 [e.g., stock prediction] or vocab_size [e.g., char-rnn])`  
![](https://github.com/gaoisbest/Tensorflow_notes_and_projects/blob/master/Q%26A_1_OutputProjectionWrapper.png)
- Reshape operations with three steps:
  - Reshape RNNs outputs from `[batch_size, time_steps, num_units]` to `[batch_size * time_steps, num_units]`.
  - Apply a fully connected layer with appropriate output size, which will result in an output with shape `[batch_size * time_steps, output_size]`.
  - Finally reshape it to `[batch_size, time_steps, output_size]`.  
![](https://github.com/gaoisbest/Tensorflow_notes_and_projects/blob/master/Q%26A_2_OutputProjection_Efficient.png)

References:  
[1] Hands on machine learning with Scikit-Learn and TensorFlow p393, p395

## 3. How to initialize RNNs weights and biases ?
The weights and biases are variables essentially. `tf.get_variable` method has a `initializer` parameter and the default is `None`. If initializer is None (the default), **the default initializer** passed in the **variable scope** will be used. Therefore, the initialization codes looks like this:

```
cell = tf.nn.rnn_cell.GRUCell(256)
with tf.variable_scope('RNN', initializer=tf.contrib.layers.xavier_initializer()):
    outputs, state = tf.nn.dynamic_rnn(cell, ...) # call dynamic_rnn will actually create the cells and their variables
```

References:  
[1] https://www.tensorflow.org/api_docs/python/tf/get_variable

## 4. How to apply dropout to RNNs ? 
`DropoutWrapper` applies dropout between the RNNs layers. Dropout should be used only during training, and `is_training` flags the status.
```
keep_prob = 0.5
cell = tf.nn.rnn_cell.GRUCell(256)
if is_training:
    cell = tf.contrib.rnn.DropoutWrapper(cell, input_keep_prob=keep_prob)
outputs, state = tf.nn.dynamic_rnn(cell, ...)
```

References:  
[1] Hands on machine learning with Scikit-Learn and TensorFlow p399  

## 5. How to build multiple layer RNNs ?
```
cells = [BasicLSTMCell(num_units) for layer in range(num_layers)]
cells_drop = [DropoutWrapper(cell, input_keep_prob=keep_prob) for cell in cells]
multi_layer_cell = MultiRNNCell(cells_drop)
```

References:  
[1] https://github.com/ageron/handson-ml/blob/master/14_recurrent_neural_networks.ipynb


# CNNs
## 1. Notes of pooling
- Similar role as Convolution layer but **without parameters**.
- **Not affect** the number of channels.
- With `2*2` kernel and stride of 2 (`tf.nn.max_pool(X, ksize=[1, 2, 2, 1], stride=[1, 2, 2, 1], padding='VALID')`), pooling will drop **75%** input values. `ksize: (batch_size, height, width , channels)` 


## 2. How to solve out-of-memory (OOV) problem during training ?
Since back propagation process requires all the intermediate values (i.e., parameters) computed during the forward pass, convolutional layer **require a huge amount of RAM (i.e., number of parameters * 32-bit floats) during training**.  
The following shows the solution:  
- Reduce the batch size.
- Large stride.
- Remove few layers.
- Try 16-bit floats.
- Distribute the model across multiple devices.

References:  
[1] Hands on machine learning with Scikit-Learn and TensorFlow p362

