---
layout: post
title: "Neural Networks Explained Simply"
date: 2026-03-27
---

## A Quick Puzzle

Look at this table for 10 seconds.

| X | Y |
|---|---|
| 1 | 3 |
| 2 | 5 |
| 3 | 7 |
| 4 | 9 |
| 5 | **?** |
| 100 | **?** |

What's Y when X is 5? What about 100?

If you said **11** and **201**, you just spotted the pattern: **Y = 2X + 1**. You looked at examples, found the relationship, and used it to predict values you'd never seen.

That's exactly what a neural network does. Except it does it with millions of inputs and patterns far too complex for any human to spot.

![Guess the pattern: X/Y table and scatter plot showing Y = 2X + 1](/assets/images/guess-the-pattern.svg)

---

## Now Make It Harder

That puzzle was easy because there was one input and a straight line. Real life isn't like that.

Imagine you're trying to predict **house prices**. The price doesn't depend on one thing -- it depends on square footage, number of bedrooms, distance to the city center, crime rate, school quality, age of the building, parking, floor number, nearby hospitals, public transport... at least 10 inputs, all interacting in complicated, non-obvious ways.

You can't eyeball a formula for that. You can't write `price = 200 * sqft` and call it done. The relationships are tangled and nonlinear. A 5th bedroom doesn't add the same value as a 2nd one. Being 1 km from the city center isn't twice as good as being 2 km away.

![From simple to complex: one input vs ten inputs feeding a neural network](/assets/images/house-price-problem.svg)

This is the problem neural networks solve: **given a pile of examples (input, output), find the hidden pattern -- no matter how complex -- so you can predict outputs for inputs you've never seen.**

You provide the examples. The network figures out the rules.

---

## So What Is a Neural Network?

A neural network is a program that **learns from examples** instead of following hand-written rules.

Traditional programming: you write the rules yourself. "If the email contains 'free money,' mark it as spam."

Neural network: you show it 10,000 emails labeled "spam" or "not spam," and it **discovers the rules on its own** -- rules so subtle you could never write them yourself.

The "neural" name comes from a loose analogy to the brain. The network is made of tiny units (neurons) connected together. But these aren't biological -- they're just small math functions. Don't overthink the name.

---

## The Building Block: A Single Neuron

Remember the puzzle? You found Y = 2X + 1. A single neuron does something similar -- it takes inputs, multiplies each by a **weight**, adds a **bias**, and produces an output.

![A single neuron with inputs, weights, bias, and activation](/assets/images/single-neuron.svg)

Here's what happens inside one neuron:

1. **Inputs arrive** -- numbers from the outside world (pixel values, house features, etc.)
2. **Each input gets multiplied by a weight** -- the weight is how important that input is. High weight = "pay attention." Negative weight = "this pushes the answer down."
3. **Add them all up, plus a bias** -- the bias shifts the answer up or down regardless of inputs (like the "+1" in Y = 2X + 1)
4. **Pass through an activation function** -- this decides if the neuron "fires" or stays quiet (explained below)
5. **Output** -- one number, passed to the next layer

**The key insight**: the weights and bias are the things the network *learns*. In Y = 2X + 1, it would learn that the weight is 2 and the bias is 1. Training is just finding the right weights.

---

## Layers: Stacking Neurons Together

One neuron can learn `Y = 2X + 1`. But it can't learn to tell a cat from a dog, or translate English to French. For that, you need many neurons organized in layers.

![Neural network with input, hidden, and output layers](/assets/images/neural-network-layers.svg)

| Layer | What it does | Example |
|-------|-------------|---------|
| **Input layer** | Receives the raw data | Pixel values, house features |
| **Hidden layers** | Discovers intermediate patterns | Edges, shapes, price factors |
| **Output layer** | Produces the final answer | "cat" at 92% confidence, or $450,000 |

Every neuron in one layer connects to every neuron in the next. Data flows **left to right**.

**Why "hidden"?** You never directly see what these layers learn. The first hidden layer might detect edges in an image. The second combines edges into shapes. The third recognizes shapes as ears, eyes, or noses. Nobody programs this -- it emerges from training.

Think of it like the house price problem: the first layer might learn "bigger houses cost more." The second might learn "but only if they're in a good neighborhood." The third might learn "unless the building is very old." Each layer captures a more nuanced relationship.

---

## Activation Functions: Learning Curves, Not Just Lines

Remember how Y = 2X + 1 is a straight line? If every neuron only did multiplication and addition, stacking 100 layers would still only produce straight lines. The math would collapse.

Activation functions fix this by adding **curves**. They let the network learn complex, non-linear patterns -- like "price goes up with size, but flattens after 3,000 sqft."

![Sigmoid, ReLU, and Tanh activation functions](/assets/images/activation-functions.svg)

| Function | What it does | When to use |
|----------|-------------|-------------|
| **Sigmoid** | Squashes any number to between 0 and 1 | Output layer for yes/no problems |
| **ReLU** | If negative, output 0. If positive, keep it. | Hidden layers (fast, simple, works great) |
| **Tanh** | Squashes to between -1 and +1 | When you need negative values |

**ReLU** is the workhorse of modern deep learning. It's dead simple -- "if negative, zero; otherwise, pass through" -- and that simplicity is why it trains fast.

---

## How It Makes Predictions (Forward Pass)

When you feed data into a trained network, it flows through every layer. This is called the **forward pass**.

![Forward pass and backward pass diagram](/assets/images/forward-backward.svg)

Walk through a concrete example -- classifying a photo as "cat" or "dog":

1. **Input**: The image becomes numbers (pixel values). A 28x28 image = 784 numbers.
2. **Hidden layer 1**: Each neuron does `(weights x inputs) + bias`, applies ReLU. This layer detects edges and textures.
3. **Hidden layer 2**: Combines edges into higher-level features -- ear shapes, fur patterns.
4. **Output layer**: Two neurons produce probabilities. Cat = 0.92, Dog = 0.08.
5. **Prediction**: Cat.

But how does it know the right weights? That's where training comes in.

---

## How It Learns (Training)

Go back to the puzzle. You had four (X, Y) examples and found the pattern. A neural network does the same thing, except the "pattern" has millions of parameters and the process is automated.

Training is a loop:

![The training loop: feed data, forward, compute loss, backward, update weights](/assets/images/training-loop.svg)

### Step 1: Guess (Forward Pass)

Feed a training example through the network. With random weights, the first guess is terrible. Maybe it says "cat = 0.5, dog = 0.5" -- a coin flip.

### Step 2: Measure the Error (Loss Function)

Compare the guess to the correct answer. The **loss function** produces a single number: "how wrong were you?"

- Network said cat = 0.3, but the answer is cat = 1.0. Loss is **high**.
- Network said cat = 0.95. Loss is **low**.

| Loss function | Used for | Idea |
|------|---------|------|
| **Mean Squared Error** | Predicting numbers (house prices) | Average of (prediction - actual)^2 |
| **Cross-Entropy** | Classification (cat vs dog) | Heavily penalizes confident wrong answers |

### Step 3: Figure Out What to Fix (Backpropagation)

The network traces backward from the loss, calculating: **how much did each weight contribute to the error?**

The intuition is simple: if increasing a weight makes the error worse, decrease it. If decreasing it makes the error worse, increase it.

### Step 4: Update the Weights (Gradient Descent)

Now you know which direction to nudge each weight. **Gradient descent** takes a small step downhill.

![Gradient descent: rolling downhill to find the best weights](/assets/images/gradient-descent.svg)

Imagine being blindfolded on a hilly landscape. You feel the slope under your feet. Step downhill. Feel again. Step again. Eventually you reach a valley -- the lowest point.

The **learning rate** controls step size:
- Too big: you overshoot the valley and bounce around
- Too small: training takes forever
- Just right: steady convergence

### Step 5: Repeat

Do this for every example. One complete pass through the data = one **epoch**. Train for tens or hundreds of epochs.

```
Epoch  1: Loss = 2.31, Accuracy = 11% (random guessing)
Epoch  5: Loss = 0.45, Accuracy = 87%
Epoch 10: Loss = 0.15, Accuracy = 95%
Epoch 20: Loss = 0.05, Accuracy = 98%
```

After 20 epochs on handwritten digit data, the network goes from random guessing (11%) to 98% accuracy. Each loop makes it slightly better.

---

## Overfitting: Memorizing Instead of Learning

A network can score 100% on training data but fail on new data. This is **overfitting** -- it memorized the examples instead of learning the pattern.

Like a student who memorizes every textbook answer but can't solve a new problem. Or fitting a crazy zigzag curve through every training point instead of finding the clean underlying trend.

| Technique | What it does |
|-----------|-------------|
| **More data** | Harder to memorize millions of examples |
| **Dropout** | Randomly turns off neurons during training -- forces the rest to pick up the slack |
| **Early stopping** | Stop when test performance starts getting worse |
| **Regularization** | Penalizes large weights, keeping the model simple |

---

## Types of Neural Networks

Different data needs different architectures.

![Feed-forward, CNN, and Transformer architectures](/assets/images/nn-types.svg)

| Type | Best for | Key idea | Examples |
|------|---------|----------|---------|
| **Feed-Forward** | Tables, simple classification | Data flows in one direction | Spam detection, price prediction |
| **CNN** | Images | Slides small filters across the image to detect patterns | Photo recognition, self-driving cars |
| **RNN / LSTM** | Sequential data | Has memory of previous inputs | Speech recognition, time series |
| **Transformer** | Text, almost everything now | Every element attends to every other | ChatGPT, translation, code |

**Transformers** have largely replaced RNNs because attention processes all words in parallel, while RNNs go one word at a time.

---

## From Neural Networks to Deep Learning

**Deep learning** = neural networks with many layers (10+). "Deep" refers to depth.

Why go deep? Each layer learns more abstract features:

```
Layer 1:  edges, gradients, color blobs
Layer 2:  textures, corners, simple shapes
Layer 3:  eyes, ears, wheels, windows
Layer 4:  faces, cars, dogs, buildings
...
Layer 50: complex concepts, abstract reasoning
```

More layers = more abstraction = more power. GPT-4 has 96+ layers.

---

## Quick Glossary

| Term | Plain English |
|------|--------------|
| **Parameters** | All the weights and biases. "7B parameters" = 7 billion adjustable numbers |
| **Epoch** | One complete pass through all training data |
| **Batch** | A small chunk processed at once (e.g., 32 images) |
| **Learning rate** | How big each weight-update step is |
| **Loss** | A number measuring how wrong the network is |
| **Gradient** | Direction of steepest descent (which way is "downhill") |
| **Inference** | Using a trained network to make predictions on new data |
| **Fine-tuning** | Training a pre-trained network further on your specific data |

---

## The Full Picture

Go back to where we started. You looked at a table, spotted a pattern, and predicted new values.

A neural network does exactly that -- at a scale and complexity no human can match. It takes millions of examples, finds patterns across thousands of dimensions, and encodes those patterns into weights.

1. **Layers of simple math units** connected by learnable weights
2. **Trained by example**: show data, measure errors, adjust weights
3. **Learned, not programmed**: patterns emerge from data, not from rules

Every "AI" you use daily -- face filters, voice assistants, recommendation engines, ChatGPT -- is a neural network that was trained on a large dataset. The concepts in this post are the foundation of all of it.

The only difference between the puzzle at the top and GPT-4? Scale. Instead of 4 data points and 2 parameters, it's trillions of data points and trillions of parameters. The core idea is the same: **look at examples, find the pattern, predict the answer**.
