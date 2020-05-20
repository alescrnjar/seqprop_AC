![SeqProp Logo](https://github.com/johli/seqprop/blob/master/SeqProp_Logo.jpg?raw=true)

# SeqProp
Stochastic Sequence Propagation - A Keras Model for optimizing DNA, RNA and protein sequences based on a predictor.

A Python API for constructing generative DNA/RNA/protein Sequence PWM models in Keras. Implements a PWM generator (with support for discrete sampling and ST gradient estimation), a predictor model wrapper and a loss model.

#### Features
- Implements a Sequence PWM Generator as a Keras Model, outputting PWMs, Logits, or random discrete samples from the PWM. These representations can be fed into any downstream Keras model for reinforcement learning.
- Implements a Predictor Keras Model wrapper, allowing easy loading of pre-trained sequence models and connecting them to the upstream PWM generator.
- Implements a Loss model with various useful cost and objectives, including regularizing PWM losses (e.g., soft sequence constraints, PWM entropy costs, etc.)
- Includes visualization code for plotting PWMs and cost functions during optimization (as Keras Callbacks).

### Installation
SeqProp can be installed by cloning or forking the [github repository](https://github.com/johli/seqprop.git):
```sh
git clone https://github.com/johli/seqprop.git
cd seqprop
python setup.py install
```

#### SeqProp requires the following packages to be installed
- Tensorflow >= 1.13.1
- Keras >= 2.2.4
- Scipy >= 1.2.1
- Numpy >= 1.16.2
- Isolearn >= 0.2.0 ([github](https://github.com/johli/isolearn.git))

### Usage
SeqProp provides API calls for building PWM generators and downstream sequence predictors as Keras Models.

A simple generator pipeline for some (imaginary) predictor can be built as follows:
```python
import keras
from keras.models import Sequential, Model, load_model
import isolearn.keras as iso
import numpy as np

from seqprop.visualization import *
from seqprop.generator import *
from seqprop.predictor import *
from seqprop.optimizer import *

from my.project import load_my_predictor #Function that loads your predictor

#Define Loss Function (Fit predicted output to some target)
#Also enforce low PWM entropy

target = np.zeros((1, 1))
target[0, 0] = 5.6 (Arbitrary target)

pwm_entropy_mse = get_target_entropy_sme(pwm_start=0, pwm_end=100, target_bits=1.8)

def loss_func(predictor_outputs) :
  pwm_logits, pwm, sampled_pwm, predicted_out = predictor_outputs
  
  #Create target constant
  target_out = K.tile(K.constant(target), (K.shape(sampled_pwm)[0], 1))
  
  target_cost = (target_out - predicted_out)**2
  pwm_cost = pwm_entropy_mse(pwm)
  
  return K.mean(target_cost + pwm_cost, axis=-1)

#Build Generator Network
_, seqprop_generator = build_generator(seq_length=100, n_sequences=1, batch_normalize_pwm=True)

#Build Predictor Network and hook it on the generator PWM output tensor
_, seqprop_predictor = build_predictor(seqprop_generator, load_my_predictor(), n_sequences=1, eval_mode='pwm')

#Build Loss Model (In: Generator seed, Out: Loss function)
_, loss_model = build_loss_model(seqprop_predictor, loss_func)

#Specify Optimizer to use
opt = keras.optimizers.Adam(lr=0.001, beta_1=0.9, beta_2=0.999)

#Compile Loss Model (Minimize self)
loss_model.compile(loss=lambda true, pred: pred, optimizer=opt)

#Fit Loss Model
loss_model.fit([], np.ones((1, 1)), epochs=1, steps_per_epoch=1000)

#Retrieve optimized PWMs and predicted (optimized) target
_, optimized_pwm, _, predicted_out = seqprop_predictor.predict(x=None, steps=1)

```

### Example Notebooks (Alternative Polyadenylation)
These examples show how to set up the PWM sequence generator model, hooking it up to a predictor, and defining various loss models. The examples build on the Alternative Polyadenylation sequence predictor APARENT.

[Notebook 1a: Generate Target Isoforms (Predict on PWM)](https://nbviewer.jupyter.org/github/johli/seqprop/blob/master/examples/apa/seqprop_aparent_isoform_optimization.ipynb)<br/>
[Notebook 1b: Generate Target Isoforms (Predict on Sampled One-hots)](https://nbviewer.jupyter.org/github/johli/seqprop/blob/master/examples/apa/seqprop_aparent_isoform_optimization_sample.ipynb)<br/>
[Notebook 2: Generate Target 3' Cleavage (Predict on Sampled One-hots)](https://nbviewer.jupyter.org/github/johli/seqprop/blob/master/examples/apa/seqprop_aparent_cleavage_optimization.ipynb)<br/>
