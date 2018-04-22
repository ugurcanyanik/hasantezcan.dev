---
layout: post
title: "Practical tutorial- LSTM neural network: A closer look under the hood"
author: "Damian Bogunowicz"
categories: blog
tags: [deep learning, neural network, tutorial, python, programming, LSTM, recurrent neural network]
image: underthehood.jpg
---

## Introduction
I have always admired LSTM networks. Currently I am working on a project, which requires a deeper understanding of the mathematical operations behind LSTM models. I have been been investigating how Keras implements this concept and sadly found out that the way it is done is not very intuitive and there are many interesting differences between the theory I have learned in the lectures and the actual implementation by the developers.
The great Richard Feynman said once: 
>What I cannot create, I do not understand.

To my mind, this means that one of the best methods to comprehend a concept, is to get our hands dirty and apply it from the scratch. By doing so, one gets a very deep and practical understanding of a concept and hopefully is able to pass on the knowledge to others. Well, let's get it started! The goal of this tutorial is to perform a foreward pass through LSTM networks using two methods. The first approach is to use a model compiled using Keras library. The second method is to read the weights of the model and implement the foreward pass on a lower level, using only numpy library. I will only scratch the surface of the maths of LSTM networks, since it has been already beautiful explained in [the blog post by Christopher Olah](http://colah.github.io/posts/2015-08-Understanding-LSTMs/). I would recommend to further read a [very elegant tutorial by Aidan Gomez](https://medium.com/@aidangomez/let-s-do-this-f9b699de31d9), where the author shows a numerical example of a foreward and backward prop in a LSTM network. 

## Architecture and the parameters of the LSTM network

A model takes a sequence of samples (observations) as an input and returns a single number (result) as an output. I call one sequence of observations a batch. Thus, a single batch is an input sequence to the network. 
Timesteps is a parameter which defines the length of a sequence. This means that the number of timesteps equals a number of samples in a batch.

Mathematically, we may say that a batch $$x$$ is a vector $$x\in \Bbb{R}^{n}$$, where $$n$$ is $$timesteps$$ parameter and the model outputs a value $$y\in \Bbb{R}$$.  

According to the classification done by Andrej Karpathy [classification done by Andrej Karpathy](http://karpathy.github.io/2015/05/21/rnn-effectiveness), we call this a many-to-one model. Let's say that $$n=timesteps=3$$. 
This means an arbitary sequence of a length three $$x=\begin{bmatrix}a\\b\\c\end{bmatrix}$$ returns a single value $$d$$ as shown below:

Finally, we define number of LSTM layers and amount of hidden units in every layer. It is assumed that every LSTM layer has the same number of hidden units. Many-to-one model requires, that after passing through all LSTM layers the values are finally processed by a single dense layer, which returns final value $$y$$. Additionally, since our input has only one feature, the dimension parameter is set to one.

## Retrieving weight matrices from the Keras model

Our next step involves extracting weights form of numpy arrays from the Keras model.

For every LSTM layer created in our Keras model, the method returns three arrays.
First array (a kernel) stores weights which correspond to $$W$$ weights for each of four LSTM gates: input gate, forget gate, cell state gate and output gate.
Second array (called a reccurent kernel) contains respective $$U$$ weights and the last, third array (bias) stores $$b$$ values.

Function <em>import_weights</em> allows us to quickly extract weights from the Keras model and store them in <em>weights_dictionary</em>, where keys of the dictionary are names of matrices and values are respective numpy arrays. Taking in account the nature of our architecture, last component of our network is a dense layer, so we additionally read off weights for that element (W and b array).
return sequences

## Defining a model in Keras

The object of the class <em>LSTM_Keras</em> simply returns a model of a neural network. We need this element to:
- obtain weights for our custom-made model
- compare results between Keras implementation and our custom-made implementation



## Defining a custom LSTM layer
The class <em>custom_LSTM</em> is the core of the script. Its task is to simulate a single LSTM layer in our network. The method <em>layer</em> is the actual implementation of the LSTM equations:

$$f_t=\sigma(W_f*x_t+U_f*h_{t-1}+b_f)$$

$$i_t=\sigma(W_i*x_t+U_i*h_{t-1}+b_i)$$

$$o_t=\sigma(W_o*x_t+U_o*h_{t-1}+b_o)$$

$$c_t={f_t}\circ{c_{t-1}}+i_t*tanh(W_c*x+U_c*h_{t-1}+b_c)$$

$${h_t=o_t * tanh(c_t)}$$

The basic functionality of the custom-made LSTM layer is 
- to change its internal state during foreward propagation of the batch
- return an intermidiate (hidden) result, which flows to the next layer.

The thing which suprised me while I was reading the Keras source code, is that an ordinary sigmoid function has been replace by a hard sigmoid. The standard sigmoid may be sometimes slow to compute because it requires calculating the exponential function. Usually the high-precision result is not needed and an approximation suffices. This is why the hard sigmoid is being used- to approximate the standard sigmoid and accelerate the computation.

Additional methods of the class allow us to reset the state  once the whole batch has passed through the layer and implement the dense layer, which gives us the final result.

As you may have already seen, our architecture is composed out of
## Implementation 

Having defined all the helper functions and classes, we can finally implement our custom-made LSTM (<em>main</em> part of the code).

Firstly, we initialize a model in Keras. The weights are automatically created using default settings (kernel weights initialized according to Xaviers initialization, recurrent kernel weights initialized as a random orthogonal matrix and bias as zero). Secondly, we create three custom-made LSTM layers. Thirdly, we create an input to the network. It is a sequence of a pre-defined size of random integers in range 0 to 100. Then, we start a loop, which computes the result of our custom-made neural network. 
To illustrate the flow of variables for a sample network, which takes batches of two samples, we may cre:


For every batch, when all samples have passed through the architecture, the last sample enters the dense layer, which computes the final output for the given batch. Additionally, the state of every LSTM layer is reset. This is to simulate what happens in Keras after each batch has been processed. By default the argument <em>stateful= False</em>. If it were True, 

> the last state for each sample a batch would be used as initial state for the first in the following batch.

Now every output $$y$$ returned by a given batch is being appended to the list <em>output_array</em>. This structure holds results returned by our network for every batch of observations. Having all results saved to one list, we may now compare the our solution with the one implemented in Keras.

## Comparison and summary

Our complicated and time-consuming implementation may be wrapped up by a single line of code in Keras, which is model.predict method. We run the code with the following parameters:

-<em>timesteps = 10 </em>        
-<em>no_of_batches = 150</em>  
-<em>no_of_layers = 3 </em>       
-<em>no_of_units = 10 </em> 

Let's take a look at the results in a form of a graph and a table.

{:refdef: style="text-align: center;"}
![alt text](https://raw.githubusercontent.com/dtransposed/dtransposed.github.io/master/assets/3/Figure_1.png "Example")
{: refdef}
<em>Figure x: Results from our implementations and Keras overlap tightly </em>


result_custom=[0.072438, 0.0699164, 0.0627664, 0.0770456, 0.0558161,...]
result_keras=[0.0724383, 0.0699164, 0.0627664, 0.0770456, 0.0558161,...]

We see that our implementation and results returned by Keras match. Since we input random numbers and process them using randomly initialized values (there was no training involved), there is also no pattern observed in our predictions. However, by replacing the Keras model, by a pre-trained network and using some more realistic input we get the following result.

## The code in Python
stateful?

<em>Source of the cover image: http://www.bleum.com</em>