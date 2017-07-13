# convgp
Code for running Gaussian processes with convolutional and symmetric structures. The code is currently being cleaned up
and will be continuously published over the next week or so.

## Paper
The accompanying paper can be found on [arXiv](https://arxiv.org/to_be_published).

## Setup
### GPflow with inter-domain support
In order to run the examples here, you need to have a branch of GPflow installed that supports inter-domain inducing
variables. The branch located [here](https://github.com/markvdw/GPflow-inter-domain) and can be installed following the
usual GPflow instructions. You can either use your favourite virtualenv to install the branch in, or switch back and
forth between the main and customised versions of GPflow by running their setup script. 

### Datasets
You will also need to setup the datasets. In `/datasets`, run:
 ```
 python process_cifar10.py
 python process_rectangles.py
 ```

## Experiments
### Speed considerations
Running on the GPU is possible, and often significantly faster when using `float32` data. This reduced precision is fine
when using stochastic optimisation, but often problematic when using (variants of) BFGS. We recommend `float32` to be
used only with stochastic optimisation, and `float64` otherwise. This has to be adjusted manually in the `gpflowrc`file.

### Running experiments
We have the following experiments:
 - `rectangles.py`: Rectangles dataset (rbf, conv, and weighted conv kernels).
 - `mnist01.py`: Zeros vs ones MNIST (rbf, conv, and weighted conv kernels).
 - `mnist.py`: Full multiclass MNIST (rbf, conv, and weighted conv kernels).
 
Many experiments have several command line options that can be used to modify a run. All have `--name`, which determines
the name of the file in `./results/` that stores the optimisation trace. Experiments are resumed if a file of the
correct name exists. Other options change the learning rate or minibatch size. See below for example experiments.

Optimisation traces can be displayed using `display.py`. The results files are passed as a positional argument, e.g.:
```
python display.py ./results/fullmnist*
```

#### Rectangles
```
python rectangles.py -k conv -M 16 --minibatch-size 100 -l 0.01 -n rectangles-paper  # Paper
python rectangles.py -k rbf -M 1200  # Compare to the optimal RBF
python rectangles.py -k wconv -M 16 --minibatch-size 100 -l 0.01  # Full solution
python rectangles.py -k conv -M 35 --minibatch-size 100 -l 0.01  # Full support on the GP
python rectangles.py -k wconv -M 35 --minibatch-size 100 -l 0.01  # Idem
python rectangles.py -k wconv -M 200 --minibatch-size 100 -l 0.01 --dataset rectangles-image --Zinit patches
```
The results for the rectangles-image dataset aren't super impressive. Need to play around with the learning rate, and
perhaps other kernels (possibly additive).

#### Mnist 0 vs 1
```
python mnist01.py -k rbf -M 20
python mnist01.py -k conv -M 50
python mnist01.py -k wconv -M 50
```

#### Mnist
```
python mnist.py -k rbf -M 750 -l 0.001  # Replicate earlier experiments with RBF kernel
python mnist.py -k conv -M 750 -l 0.001 --minibatch-size 200
python mnist.py -k wconv -M 750 -l 0.001 --minibatch-size 200
```

## Reproducing the plots from the paper
After running the above experiments, you can run `python paper-plots.py` to recreate the figures from the paper.

## Notes on the code
While the repositories for `gpflow-inter-domain` and `convgp` are separate, they rely on some modifications in each
other. The most non-elegant adaptation to GPflow is to allow variables internal to the TensorFlow optimiser to be
restored through opt_tools. The whole set up is a bit less than ideal, it would probably be better to use the internal
TensorFlow loading and storing mechanisms, but this would require larger edits to GPflow.