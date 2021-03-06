# MIT-6.S191-Introduction-to-Deep-Learning
- Below are my personal notes after taking the online MIT Introduction to Deep Learning course.
- Course content can be found [here](http://introtodeeplearning.com/).

## Introduction to Deep Learning
- All activation functions are non-linear.
- The purpose of activation functions is to **introduce non-linearities** into the network.
- Common activation functions:
  - Sigmoid function: `tf.nn.sigmoid(z)`: 
    - *g(z) = (1 + e<sup>-z</sup>)<sup>-1</sup>*
    - *g<sup>'</sup>(z) = g(z) × (1 - g(z))*
  - Hyperbolic Tangent: `tf.nn.tanh(z)`: 
    - *g(z) = (e<sup>z</sup> - e<sup>-z</sup>) × (e<sup>z</sup> + e<sup>-z</sup>)<sup>-1</sup>*
    - *g<sup>'</sup>(z) = 1 - g(z)<sup>2</sup>*
  - Rectified Linear Unit (ReLU):
    - *g(z) = max(0, z)*
    - *g<sup>'</sup>(z) = 1* when *z > 0* and *0* otherwise.
- The **loss** of our network measures the cost incurred from incorrect predictions.
  - *L(f(x<sup>(i)</sup>; **W**), y<sup>(i)</sup>)*
- The **empirical loss** measures the total loss over our entire dataset.
  - ***J(W)** = n<sup>-1</sup>**Σ**L(f(x<sup>(i)</sup>; **W**), y<sup>(i)</sup>)*
- **Cross entropy loss** can be used with models that output a probability between 0 and 1.
  - ***J(W)** = n<sup>-1</sup>**Σ**y<sup>(i)</sup>log(f(x<sup>(i)</sup>; **W**)) + (1 - y<sup>(i)</sup>)log(1 - f(x<sup>(i)</sup>; **W**))*
  - `loss = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(model.y, model.pred))`
- **Mean squared error loss** can be used with regression models that output continuous real numbers.
  - ***J(W)** = n<sup>-1</sup>**Σ**(y<sup>(i)</sup> - f(x<sup>(i)</sup>; **W**))<sup>2</sup>*
  - `loss = tf.reduce_mean(tf.square(tf.subtract(model.y, model.pred))`
- **Loss optimisation** is to find the network weights that **achieve the lowest loss**.
  - ***W**<sup>\*</sup> = argmin J(**W**)*
- Gradient Descent:
  - Initialise weights randomly: `weights = tf.random_normal(shape, stddev=sigma)`
  - Loop until convergence:
    - Compute gradient: `grads = tf.gradients(ys=loss, xs=weights)`
    - Update weights: `weights_new = weights.assign(weights - learning_rate * grads)`
  - Return weights.
- **Small learning rates** converge slowly and get stuck in false local minima.
- **Large learning rates** overshoot, become unstable and diverge.
- **Stable learning rates** converge smoothly and avoid local minima.
- Adaptive Learning Rates:
  - Learning rates are no longer fixed.
  - Can be made larger or smaller depending on:
    - how large gradient is.
    - how fast learning is happening.
    - size of particular weights.
    - etc.
- Algorithm for Adaptive Learning Rates:
  - Momentum: `tf.train.MomentumOptimizer`
  - Adagrad: `tf.train.AdagradOptimizer`
  - Adadelta: `tf.train.AdadeltaOptimizer`
  - Adam: `tf.train.AdamOptimizer`
  - RMSProp: `tf.train.RMSPropOptimizer`
- **Mini-batches** while training leads to:
  - More accurate estimation of gradient: smoother convergence and possible to have larger learning rates.
  - Fast training: parallel computation and significant speed increases on GPU's.
- **Regularisation** is a technique that contrains our optimisation problem to discourage complex models. This technique helps improve generalisation of the model on unseen data.
  - Reguralisation 1: Dropout: 
    - During training, randomly set some activations to 0. This forces network to not rely on any single node.
    - `tf.keras.layers.Dropout(p=0.5)`
  - Regularisation 2: Early Stopping:
    - Stop training before we have a chance to overfit.

## Deep Sequence Modeling with Recurrent Neural Networks (RNN)
- Problems when modelling sequence:
  - Problem 1: A fixed window can't model long-term dependencies.
  - Problem 2: Set of counts (Bag of words) doesn't preserve order.
  - Problem 3: Big fixed window doesn't enable parameter sharing.
- To model sequences, we need to:
  - Handle **variable-length** sequences.
  - Track **long-term** dependencies.
  - Maintain information about **order**.
  - **Share parameters** across the sequence.
- RNN can model sequences well.
- One to One: "Vanilla" Neural Network.
- Many to One: Sentiment Classification.
- Many to Many: Music Generation.
- RNN applies a recurrence relation at every time step to process a sequence.
  - *h<sub>t</sub> = f<sub>W</sub>(h<sub>t-1</sub>, x<sub>t</sub>)*
  - The same function and set of parameters are used at every time step.
- RNN state update and output:
  - Output vector: *y_hat = **W**<sub>hy</sub>h<sub>t</sub>*
  - Update hidden state: *h<sub>t</sub> = tanh(**W**<sub>hh</sub>h<sub>t-1</sub> + **W**<sub>xh</sub>x<sub>t</sub>)*
- Computing the gradient wrt *h<sub>0</sub>* involves **many factors of *W<sub>hh</sub>*** (and repeated *f<sup>'</sup>*!).
  - Exploding Gradients ← Gradient clipping to scale big gradients.
  - Vanishing Gradients: Multiplying many **small numbers** together → Errors due to further back time steps have smaller and smaller gradients → Bias network to capture short-term dependencies.
    - Trick 1: Activation functions: ReLU prevents ***f<sup>'</sup>*** from shrinking the gradients when *x > 0*.
    - Trick 2: Initilising weights to identity matrix and biases to zero helps prevent the weights from shrinking to zero.
    - Trick 3: Gated cells (LSTM, GRU, etc.: more complex recurrent unit with gates to control what information is passed through).
- **Long Short Term Memory (LSTMs)** networks rely on a gated cell to track information throughout many steps.
  - LSTM repeating modules contain **interacting layers** that **control information flow**, while in a standard RNN, repeating modules contain a **simple computation node**.
  - LSTM's main a **cell state *C<sub>t</sub>*** where it's easy for information to flow. 
  - Information is **added** or **removed** to cell state through structures called **gates**.
  - LSTM's **forget irrelevant** parts of the previous state.
    - *f<sub>t</sub> = σ(**W<sub>i</sub>**\[h<sub>t-1</sub>, x<sub>t</sub>\] + b<sub>f</sub>)*
    - Use previous cell output and input
    - Sigmoid: value 0 and 1 indicate "completely forget" and "permanent keep" respectively.
  - LSTM's **selectively update** cell state values.
    - *i<sub>t</sub> = σ(**W<sub>i</sub>**\[h<sub>t-1</sub>, x<sub>t</sub>\] + b<sub>i</sub>)*
    - *C~<sub>t</sub> = tanh(**W<sub>C</sub>**\[h<sub>t-1</sub>, x<sub>t</sub>\] + b<sub>C</sub>)*
    - Sigmoid layer: decide what values to update.
    - Tanh layer: generate new vector of "candidate values" that could be added to the state.
    - *C<sub>t</sub> = f<sub>t</sub> × C<sub>t-1</sub> + i<sub>t</sub> × C~<sub>t</sub>*
  - LTSM's use an **output gate** to output certain parts of the cell state.
    - *o<sub>t</sub> = σ(**W<sub>o</sub>**\[h<sub>t-1</sub>, x<sub>t</sub>\] + b<sub>o</sub>)*
    - *h<sub>t</sub> = o<sub>t</sub> × tanh(C<sub>t</sub>)*
    - Sigmoid layer: decide what parts of state to output.
    - Tanh layer: squash values between -1 and 1.
    - *o<sub>t</sub> × tanh(C<sub>t</sub>)*: output filtered version of cell state.
  - Uninterrupted gradient flow: Backpropagation from ***C<sub>t</sub>*** to ***C<sub>t-1</sub>*** requires only elementwise multiplication which avoids vanishing gradient problem.

## Deep Reinforcement Learning
- Classes of Learning Problems:
  
  ||Supervised Learning|Unsupervised Learning|Reinforcement Learning|
  |:-|:-|:-|:-|
  |Data|(x,y). x is data, y is label|x. No Label|state-action pairs|
  |Goal|Map x → y|Learn underlying structure|Maximise future rewards over many time steps|
  |Apple example| This thing is an apple|This thing is like the other thing|Eat this thing because it will keep you alive|
- Reinforcement Learning Key Concepts:
  - **Agent**: takes actions.
  - **Environment**: the world in which the agent exists and operates.
  - **Action**: a move the agent can make in the environment.
  - **Observation**: of the environment after taking actions.
  - **State**: a situation which the agent perceives.
  - **Reward**: feedback that measures the success or failure of the agent's action.
  - **Total Reward**: *R<sub>t</sub> = Σr<sub>i</sub>*. 
  - Because *R<sub>t</sub>* can go to infinity, there has to be a **discount factor** which places more weight on short-term timestamp reward and less weight on long-term timestamp from the current state reward. Thus, we have *R<sub>t</sub> = Σγ<sup>i</sup>r<sub>i</sub>*.
  - **Q-function** captures the **expected total future reward** an agent in state, *s*, can receive by executing a certain action, *a*. Thus, we have *Q(s,a) = E[R<sub>t</sub>]*
  - The agent needs a **policy π(s)**, to infer the **best action to take** at its state, *s*.
  - **Strategy**: the policy should choose an action that maximises future reward. *π\*(s) = argmax Q(s,a)*.
- Deep Reinforcement Learning Algorithms:
  - **Value learning**: Find *Q(s,a). a = argmax Q(s,a)*.
  - **Policy learning**: Find *π(s)*. Sample *a ~ π(s)*.
- **Deep Q Networks (DQN)**:
  
  |Input|NN|Output|
  |:-:|:-:|:-:|
  |<ul><li>state, *s*</li><li>action, *a*</li></ul>|Deep NN|*Q(s,a)*|
  |state, *s*|Deep NN|<ul><li>*Q(s,a<sub>1</sub>)*</li><li>*Q(s,a<sub>2</sub>)*</li></ul>|
- Downsides of Q-learning:
  - Complexity: 
    - Can model scenarios where the action space is discrete and small.
    - Cannot handle continuous action spaces.
  - Flexibility:
    - Cannot learn stochastic policies since policy is deterministically computed from the Q function.
- **Policy Gradient**: directly optimises the policy, while DQN tries to approximate Q and infer the optimal policy.
  - Run a policy for a while.
  - Increase probability of actions that lead to high rewards.
  - Decrease the probability of actions that lead to low/no rewards.

## Deep Learning Limitations and New Frontiers
- Neural Network Limitations:
  - Very **data hungry** (eg. often millions of examples).
  - **Computationally intensive** to train and deploy (tractably requires GPUs).
  - Easily fooled by **adversarial examples**.
  - Can be subject to **algorithmic bias**.
  - Poor at **representing uncertainty** (how do you know what the model knows?).
  - Uninterpretable **black boxes**, difficult to trust.
  - **Finicky to optimize**: non-convex, choice of architecture, learning parameters.
  - Often require **expert knowledge** to design, fine tune architectures.
- New Frontiers:
  - Bayesian Deep Learning for Uncertainty.
  - Learning to Learn. Google AutoML:
    - Design an AI algorithm that can build new models capable of solving a task.
    - Reduce the need for the experienced engineers to design the network.
    - Make deep learning more accessible to the public.
