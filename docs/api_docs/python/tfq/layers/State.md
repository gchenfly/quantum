<div itemscope itemtype="http://developers.google.com/ReferenceObject">
<meta itemprop="name" content="tfq.layers.State" />
<meta itemprop="path" content="Stable" />
<meta itemprop="property" content="__call__"/>
<meta itemprop="property" content="__init__"/>
<meta itemprop="property" content="build"/>
<meta itemprop="property" content="compute_mask"/>
<meta itemprop="property" content="compute_output_shape"/>
<meta itemprop="property" content="count_params"/>
<meta itemprop="property" content="from_config"/>
<meta itemprop="property" content="get_config"/>
<meta itemprop="property" content="get_input_at"/>
<meta itemprop="property" content="get_input_mask_at"/>
<meta itemprop="property" content="get_input_shape_at"/>
<meta itemprop="property" content="get_losses_for"/>
<meta itemprop="property" content="get_output_at"/>
<meta itemprop="property" content="get_output_mask_at"/>
<meta itemprop="property" content="get_output_shape_at"/>
<meta itemprop="property" content="get_updates_for"/>
<meta itemprop="property" content="get_weights"/>
<meta itemprop="property" content="set_weights"/>
<meta itemprop="property" content="with_name_scope"/>
</div>

# tfq.layers.State

<!-- Insert buttons and diff -->

<table class="tfo-notebook-buttons tfo-api" align="left">

<td>
  <a target="_blank" href="https://github.com/tensorflow/quantum/tree/master/tensorflow_quantum/python/layers/circuit_executors/state.py">
    <img src="https://www.tensorflow.org/images/GitHub-Mark-32px.png" />
    View source on GitHub
  </a>
</td></table>



A Layer that simulates a quantum state.

```python
tfq.layers.State(
    backend=None, **kwargs
)
```



<!-- Placeholder for "Used in" -->

Given an input circuit and set of parameter values, Simulate a quantum state
and output it to the Tensorflow graph.


A more common application is for determining the set of states produced
by a parametrized circuit where the values of the parameters vary. Suppose
we want to generate a family of states with varying degrees of entanglement
ranging from separable to maximally entangled. We first define a
parametrized circuit that can accomplish this

```
>>> q0, q1 = cirq.GridQubit.rect(1, 2)
>>> alpha = sympy.Symbol('alpha') # degree of entanglement between q0, q1
>>> parametrized_bell_circuit = cirq.Circuit(
...    cirq.H(q0), cirq.CNOT(q0, q1) ** alpha)
```

Now pass all of the alpha values desired to <a href="../../tfq/layers/State.md"><code>tfq.layers.State</code></a> to compute
a tensor of states corresponding to these preparation angles.

```
>>> state_layer = tfq.layers.State()
>>> alphas = tf.reshape(tf.range(0, 1.1, delta=0.5), (3, 1)) # FIXME: #805
>>> state_layer(parametrized_bell_circuit,
...     symbol_names=[alpha], symbol_values=alphas)
<tf.RaggedTensor [[0.707106, 0j, 0.707106, 0j],
[(0.707106-1.2802768623032534e-08j), 0j,
    (0.353553+0.3535534143447876j), (0.353553-0.3535533547401428j)],
[(0.707106-1.2802768623032534e-08j), 0j,
    (0.-3.0908619663705394e-08j), (0.707106+6.181723932741079e-08j)]]>
```


This use case can be simplified to compute the wavefunction produced by a
fixed circuit where the values of the parameters vary. For example, this
layer produces a Bell state.

```
>>> q0, q1 = cirq.GridQubit.rect(1, 2)
>>> bell_circuit = cirq.Circuit(cirq.H(q0), cirq.CNOT(q0, q1))
>>> state_layer = tfq.layers.State()
>>> state_layer(bell_circuit)
<tf.RaggedTensor [[(0.707106-1.2802768623032534e-08j),
                    0j,
                   (0.-3.0908619663705394e-08j),
                   (0.707106+6.181723932741079e-08j)]]>
```

Not specifying `symbol_names` or `symbol_values` indicates that the
circuit(s) does not contain any `sympy.Symbols` inside of it and tfq won't
look for any symbols to resolve.


<a href="../../tfq/layers/State.md"><code>tfq.layers.State</code></a> also allows for a more complicated input signature
wherein a different (possibly parametrized) circuit is used to prepare
a state for each batch of input parameters. This might be useful when
the State layer is being used to generate entirely different families
of states. Suppose we want to generate a stream of states that are
either computational basis states or 'diagonal' basis states (as in the
BB84 QKD protocol). The circuits to prepare these states are:

```
>>> q0 = cirq.GridQubit(0, 0)
>>> bitval = sympy.Symbol('bitval')
>>> computational_circuit = cirq.Circuit(cirq.X(q0) ** bitval)
>>> diagonal_circuit = cirq.Circuit(cirq.X(q0) ** bitval, cirq.H(q0))
```

Now a stream of random classical bit values can be encoded into one of
these bases by preparing a state layer and passing in the bit values
accompanied by their preparation circuits

```
>>> qkd_layer = tfq.layers.State()
>>> bits = [[1], [1], [0], [0]]
>>> states_to_send = [computational_circuit,
...                   diagonal_circuit,
...                   diagonal_circuit,
...                   computational_circuit]
>>> qkd_states = qkd_layer(
...     states_to_send, symbol_names=[bitval], symbol_values=bits)
>>> # The third state was a '0' prepared in the diagonal basis:
>>> qkd_states
<tf.RaggedTensor [[-4.371138828673793e-08j, (1+4.371138828673793e-08j)],
[(0.707106+3.0908619663705394e-08j), (-0.707106-1.364372508305678e-07j)],
[(0.707106-1.2802768623032534e-08j), (0.707106+3.0908619663705394e-08j)],
[(1+0j), 0j]]>
```


Note: When specifying a new layer for a *compiled* `tf.keras.Model` using
something like `tfq.layers.State()(cirq.Circuit(...), ...)` please
be sure to instead use `tfq.layers.State()(circuit_input, ...)`
where `circuit_input` is a `tf.keras.Input` that is filled with
`tfq.conver_to_tensor([cirq.Circuit(..)] * batch_size)` at runtime. This
is because compiled keras models require non keyword layer `call` inputs to
be traceable back to a `tf.keras.Input`.

#### Args:


* <b>`backend`</b>: Optional Backend to use to simulate this state. Defaults
    to the native TensorFlow Quantum state vector simulator,
    however users may also specify a preconfigured cirq execution
    object to use instead, which must inherit
    `cirq.SimulatesFinalState`. Note that C++ Density Matrix
    simulation is not yet supported so to do Density Matrix
    simulation please use `cirq.DensityMatrixSimulator`.

#### Attributes:

* <b>`activity_regularizer`</b>:   Optional regularizer function for the output of this layer.
* <b>`dtype`</b>
* <b>`dynamic`</b>
* <b>`input`</b>:   Retrieves the input tensor(s) of a layer.

  Only applicable if the layer has exactly one input,
  i.e. if it is connected to one incoming layer.

* <b>`input_mask`</b>:   Retrieves the input mask tensor(s) of a layer.

  Only applicable if the layer has exactly one inbound node,
  i.e. if it is connected to one incoming layer.

* <b>`input_shape`</b>:   Retrieves the input shape(s) of a layer.

  Only applicable if the layer has exactly one input,
  i.e. if it is connected to one incoming layer, or if all inputs
  have the same shape.

* <b>`input_spec`</b>
* <b>`losses`</b>:   Losses which are associated with this `Layer`.

  Variable regularization tensors are created when this property is accessed,
  so it is eager safe: accessing `losses` under a `tf.GradientTape` will
  propagate gradients back to the corresponding variables.
* <b>`metrics`</b>
* <b>`name`</b>:   Returns the name of this module as passed or determined in the ctor.

  NOTE: This is not the same as the `self.name_scope.name` which includes
  parent module names.
* <b>`name_scope`</b>:   Returns a `tf.name_scope` instance for this class.
* <b>`non_trainable_variables`</b>
* <b>`non_trainable_weights`</b>
* <b>`output`</b>:   Retrieves the output tensor(s) of a layer.

  Only applicable if the layer has exactly one output,
  i.e. if it is connected to one incoming layer.

* <b>`output_mask`</b>:   Retrieves the output mask tensor(s) of a layer.

  Only applicable if the layer has exactly one inbound node,
  i.e. if it is connected to one incoming layer.

* <b>`output_shape`</b>:   Retrieves the output shape(s) of a layer.

  Only applicable if the layer has one output,
  or if all outputs have the same shape.

* <b>`submodules`</b>:   Sequence of all sub-modules.

  Submodules are modules which are properties of this module, or found as
  properties of modules which are properties of this module (and so on).

  ```
  a = tf.Module()
  b = tf.Module()
  c = tf.Module()
  a.b = b
  b.c = c
  assert list(a.submodules) == [b, c]
  assert list(b.submodules) == [c]
  assert list(c.submodules) == []
  ```
* <b>`trainable`</b>
* <b>`trainable_variables`</b>:   Sequence of trainable variables owned by this module and its submodules.

  Note: this method uses reflection to find variables on the current instance
  and submodules. For performance reasons you may wish to cache the result
  of calling this method if you don't expect the return value to change.
* <b>`trainable_weights`</b>
* <b>`updates`</b>
* <b>`variables`</b>:   Returns the list of all layer variables/weights.

  Alias of `self.weights`.
* <b>`weights`</b>:   Returns the list of all layer variables/weights.



## Methods

<h3 id="__call__"><code>__call__</code></h3>

```python
__call__(
    inputs, *args, **kwargs
)
```

Wraps `call`, applying pre- and post-processing steps.


#### Arguments:


* <b>`inputs`</b>: input tensor(s).
* <b>`*args`</b>: additional positional arguments to be passed to `self.call`.
* <b>`**kwargs`</b>: additional keyword arguments to be passed to `self.call`.


#### Returns:

Output tensor(s).



#### Note:

- The following optional keyword arguments are reserved for specific uses:
  * `training`: Boolean scalar tensor of Python boolean indicating
    whether the `call` is meant for training or inference.
  * `mask`: Boolean input mask.
- If the layer's `call` method takes a `mask` argument (as some Keras
  layers do), its default value will be set to the mask generated
  for `inputs` by the previous layer (if `input` did come from
  a layer that generated a corresponding mask, i.e. if it came from
  a Keras layer with masking support.



#### Raises:


* <b>`ValueError`</b>: if the layer's `call` method returns None (an invalid value).

<h3 id="build"><code>build</code></h3>

```python
build(
    input_shape
)
```

Creates the variables of the layer (optional, for subclass implementers).

This is a method that implementers of subclasses of `Layer` or `Model`
can override if they need a state-creation step in-between
layer instantiation and layer call.

This is typically used to create the weights of `Layer` subclasses.

#### Arguments:


* <b>`input_shape`</b>: Instance of `TensorShape`, or list of instances of
  `TensorShape` if the layer expects a list of inputs
  (one instance per input).

<h3 id="compute_mask"><code>compute_mask</code></h3>

```python
compute_mask(
    inputs, mask=None
)
```

Computes an output mask tensor.


#### Arguments:


* <b>`inputs`</b>: Tensor or list of tensors.
* <b>`mask`</b>: Tensor or list of tensors.


#### Returns:

None or a tensor (or list of tensors,
    one per output tensor of the layer).


<h3 id="compute_output_shape"><code>compute_output_shape</code></h3>

```python
compute_output_shape(
    input_shape
)
```

Computes the output shape of the layer.

If the layer has not been built, this method will call `build` on the
layer. This assumes that the layer will later be used with inputs that
match the input shape provided here.

#### Arguments:


* <b>`input_shape`</b>: Shape tuple (tuple of integers)
    or list of shape tuples (one per output tensor of the layer).
    Shape tuples can include None for free dimensions,
    instead of an integer.


#### Returns:

An input shape tuple.


<h3 id="count_params"><code>count_params</code></h3>

```python
count_params()
```

Count the total number of scalars composing the weights.


#### Returns:

An integer count.



#### Raises:


* <b>`ValueError`</b>: if the layer isn't yet built
  (in which case its weights aren't yet defined).

<h3 id="from_config"><code>from_config</code></h3>

```python
@classmethod
from_config(
    cls, config
)
```

Creates a layer from its config.

This method is the reverse of `get_config`,
capable of instantiating the same layer from the config
dictionary. It does not handle layer connectivity
(handled by Network), nor weights (handled by `set_weights`).

#### Arguments:


* <b>`config`</b>: A Python dictionary, typically the
    output of get_config.


#### Returns:

A layer instance.


<h3 id="get_config"><code>get_config</code></h3>

```python
get_config()
```

Returns the config of the layer.

A layer config is a Python dictionary (serializable)
containing the configuration of a layer.
The same layer can be reinstantiated later
(without its trained weights) from this configuration.

The config of a layer does not include connectivity
information, nor the layer class name. These are handled
by `Network` (one layer of abstraction above).

#### Returns:

Python dictionary.


<h3 id="get_input_at"><code>get_input_at</code></h3>

```python
get_input_at(
    node_index
)
```

Retrieves the input tensor(s) of a layer at a given node.


#### Arguments:


* <b>`node_index`</b>: Integer, index of the node
    from which to retrieve the attribute.
    E.g. `node_index=0` will correspond to the
    first time the layer was called.


#### Returns:

A tensor (or list of tensors if the layer has multiple inputs).



#### Raises:


* <b>`RuntimeError`</b>: If called in Eager mode.

<h3 id="get_input_mask_at"><code>get_input_mask_at</code></h3>

```python
get_input_mask_at(
    node_index
)
```

Retrieves the input mask tensor(s) of a layer at a given node.


#### Arguments:


* <b>`node_index`</b>: Integer, index of the node
    from which to retrieve the attribute.
    E.g. `node_index=0` will correspond to the
    first time the layer was called.


#### Returns:

A mask tensor
(or list of tensors if the layer has multiple inputs).


<h3 id="get_input_shape_at"><code>get_input_shape_at</code></h3>

```python
get_input_shape_at(
    node_index
)
```

Retrieves the input shape(s) of a layer at a given node.


#### Arguments:


* <b>`node_index`</b>: Integer, index of the node
    from which to retrieve the attribute.
    E.g. `node_index=0` will correspond to the
    first time the layer was called.


#### Returns:

A shape tuple
(or list of shape tuples if the layer has multiple inputs).



#### Raises:


* <b>`RuntimeError`</b>: If called in Eager mode.

<h3 id="get_losses_for"><code>get_losses_for</code></h3>

```python
get_losses_for(
    inputs
)
```

Retrieves losses relevant to a specific set of inputs.


#### Arguments:


* <b>`inputs`</b>: Input tensor or list/tuple of input tensors.


#### Returns:

List of loss tensors of the layer that depend on `inputs`.


<h3 id="get_output_at"><code>get_output_at</code></h3>

```python
get_output_at(
    node_index
)
```

Retrieves the output tensor(s) of a layer at a given node.


#### Arguments:


* <b>`node_index`</b>: Integer, index of the node
    from which to retrieve the attribute.
    E.g. `node_index=0` will correspond to the
    first time the layer was called.


#### Returns:

A tensor (or list of tensors if the layer has multiple outputs).



#### Raises:


* <b>`RuntimeError`</b>: If called in Eager mode.

<h3 id="get_output_mask_at"><code>get_output_mask_at</code></h3>

```python
get_output_mask_at(
    node_index
)
```

Retrieves the output mask tensor(s) of a layer at a given node.


#### Arguments:


* <b>`node_index`</b>: Integer, index of the node
    from which to retrieve the attribute.
    E.g. `node_index=0` will correspond to the
    first time the layer was called.


#### Returns:

A mask tensor
(or list of tensors if the layer has multiple outputs).


<h3 id="get_output_shape_at"><code>get_output_shape_at</code></h3>

```python
get_output_shape_at(
    node_index
)
```

Retrieves the output shape(s) of a layer at a given node.


#### Arguments:


* <b>`node_index`</b>: Integer, index of the node
    from which to retrieve the attribute.
    E.g. `node_index=0` will correspond to the
    first time the layer was called.


#### Returns:

A shape tuple
(or list of shape tuples if the layer has multiple outputs).



#### Raises:


* <b>`RuntimeError`</b>: If called in Eager mode.

<h3 id="get_updates_for"><code>get_updates_for</code></h3>

```python
get_updates_for(
    inputs
)
```

Retrieves updates relevant to a specific set of inputs.


#### Arguments:


* <b>`inputs`</b>: Input tensor or list/tuple of input tensors.


#### Returns:

List of update ops of the layer that depend on `inputs`.


<h3 id="get_weights"><code>get_weights</code></h3>

```python
get_weights()
```

Returns the current weights of the layer.


#### Returns:

Weights values as a list of numpy arrays.


<h3 id="set_weights"><code>set_weights</code></h3>

```python
set_weights(
    weights
)
```

Sets the weights of the layer, from Numpy arrays.


#### Arguments:


* <b>`weights`</b>: a list of Numpy arrays. The number
    of arrays and their shape must match
    number of the dimensions of the weights
    of the layer (i.e. it should match the
    output of `get_weights`).


#### Raises:


* <b>`ValueError`</b>: If the provided weights list does not match the
    layer's specifications.

<h3 id="with_name_scope"><code>with_name_scope</code></h3>

```python
@classmethod
with_name_scope(
    cls, method
)
```

Decorator to automatically enter the module name scope.

```
class MyModule(tf.Module):
  @tf.Module.with_name_scope
  def __call__(self, x):
    if not hasattr(self, 'w'):
      self.w = tf.Variable(tf.random.normal([x.shape[1], 64]))
    return tf.matmul(x, self.w)
```

Using the above module would produce `tf.Variable`s and `tf.Tensor`s whose
names included the module name:

```
mod = MyModule()
mod(tf.ones([8, 32]))
# ==> <tf.Tensor: ...>
mod.w
# ==> <tf.Variable ...'my_module/w:0'>
```

#### Args:


* <b>`method`</b>: The method to wrap.


#### Returns:

The original method wrapped such that it enters the module's name scope.




