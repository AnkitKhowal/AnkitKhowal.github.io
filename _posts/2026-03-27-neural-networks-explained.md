---
layout: post
title: "Neural Networks Explained Simply"
date: 2026-03-27
---

Neural networks power everything from face filters on your phone to ChatGPT. They sound complicated, but the core ideas are surprisingly intuitive. This post explains every concept from the ground up — no math background required.

---

## What Is a Neural Network?

A neural network is a program that **learns from examples** instead of being explicitly programmed.

Traditional programming: you write rules. "If the email contains 'free money,' mark it as spam."

Neural network: you show it thousands of emails labeled "spam" or "not spam," and it **figures out the rules on its own**.

The "neural" part comes from a loose analogy to the brain — the network is made of small, simple units (neurons) connected together. But don't take the analogy too literally. These are math functions, not biology.

---

## The Building Block: A Single Neuron

A neuron does one simple thing: it takes some numbers in, does a tiny bit of math, and produces one number out.

![A single neuron with inputs, weights, bias, and activation](/assets/images/single-neuron.svg)

Here's what happens step by step:

1. **Inputs arrive** — numbers from the outside world (like pixel values from an image)
2. **Each input gets multiplied by a weight** — weights represent how important each input is. A high weight means "pay attention to this." A negative weight means "this pushes the answer down."
3. **Add them all up, plus a bias** — the bias is like a baseline. It shifts the answer up or down regardless of the inputs.
4. **Pass through an activation function** — this decides whether the neuron "fires" or stays quiet (more on this below)
5. **Output** — a single number that gets passed to the next layer

**The key insight**: the weights and bias are the things the network *learns*. Training is just the process of finding the right weights.

---

## Layers: Stacking Neurons Together

One neuron can't do much. But stack hundreds of them in layers, and you get something powerful.

![Neural network with input, hidden, and output layers](/assets/images/neural-network-layers.svg)

A neural network has three types of layers:

| Layer | What it does | Example |
|-------|-------------|---------|
| **Input layer** | Receives the raw data | Pixel values of an image |
| **Hidden layers** | Discovers patterns in the data | Edges → shapes → faces |
| **Output layer** | Produces the final answer | "cat" with 92% confidence |

Every neuron in one layer connects to every neuron in the next. Data flows **left to right** — input to output. That's why it's called a "feed-forward" network.

**Why "hidden"?** Because you never directly see what these layers do. They figure out intermediate patterns on their own. The first hidden layer might detect edges. The second might combine edges into shapes. The third might recognize shapes as ears, eyes, or noses. Nobody programs this — it emerges from training.

---

## Activation Functions: Why Non-Linearity Matters

If neurons only did addition and multiplication (linear math), stacking 100 layers would be no better than having one layer. The math would just collapse.

Activation functions break this by adding **non-linearity** — they let the network learn curves, not just straight lines.

![Sigmoid, ReLU, and Tanh activation functions](/assets/images/activation-functions.svg)

The three most common ones:

| Function | What it does | When to use |
|----------|-------------|-------------|
| **Sigmoid** | Squashes any number to between 0 and 1 | Output layer for yes/no problems |
| **ReLU** | If negative → 0. If positive → keep it. | Hidden layers (fast, simple, works great) |
| **Tanh** | Squashes to between -1 and +1 | When you need negative values |

**ReLU** is the workhorse of modern deep learning. It's dead simple — "if negative, zero; otherwise, pass through unchanged" — and that simplicity is why it trains fast.

---

## How a Network Makes Predictions (Forward Pass)

When you feed data into a trained network, it flows through every layer from left to right. This is called the **forward pass**.

![Forward pass and backward pass diagram](/assets/images/forward-backward.svg)

Let's walk through a concrete example — classifying a photo as "cat" or "dog":

1. **Input**: The image is converted into numbers (pixel values). A tiny 28×28 grayscale image becomes 784 numbers.
2. **Hidden layer 1**: Each neuron computes `(weights × inputs) + bias`, applies ReLU. Maybe this layer detects edges and textures.
3. **Hidden layer 2**: Takes the edge/texture signals and combines them into higher-level features — ears, fur patterns, snout shapes.
4. **Output layer**: Two neurons, one for "cat" and one for "dog." They use sigmoid so each outputs a probability between 0 and 1.
5. **Result**: Cat = 0.92, Dog = 0.08. The network predicts "cat."

But how does it know the right weights? That's where training comes in.

---

## How a Network Learns (Training)

Training is a loop. You show the network examples, measure how wrong it is, and adjust the weights to be less wrong. Repeat thousands of times.

![The training loop: feed data, forward, compute loss, backward, update weights](/assets/images/training-loop.svg)

### Step 1: Make a prediction (Forward Pass)

Feed a training example (e.g., a photo of a cat labeled "cat") through the network. It outputs a prediction — probably wrong at first, since the weights are random.

### Step 2: Measure the error (Loss Function)

Compare the prediction to the correct answer. The **loss function** produces a single number that says "how wrong were you?"

- If the network said "cat = 0.3" but the answer is "cat = 1.0," the loss is high.
- If it said "cat = 0.95," the loss is low.

Common loss functions:

| Name | Used for | Idea |
|------|---------|------|
| **Mean Squared Error** | Predicting numbers (regression) | Average of (prediction - actual)² |
| **Cross-Entropy** | Classification (cat vs dog) | Penalizes confident wrong answers heavily |

### Step 3: Figure out what to fix (Backward Pass / Backpropagation)

This is the clever part. The network traces backward from the loss through every layer, calculating: **"how much did each weight contribute to the error?"**

This uses calculus (specifically the chain rule), but the intuition is simple: if increasing a weight makes the error worse, decrease it. If decreasing it makes the error worse, increase it.

### Step 4: Update the weights (Gradient Descent)

Now you know which direction to nudge each weight. **Gradient descent** takes a small step in the direction that reduces the loss.

![Gradient descent: rolling downhill to find the best weights](/assets/images/gradient-descent.svg)

Think of it like being blindfolded on a hilly landscape. You can feel the slope under your feet. Step downhill. Feel again. Step again. Eventually you reach a valley — the point where the loss is lowest.

The **learning rate** controls how big each step is:
- Too big → you overshoot the valley and bounce around
- Too small → training takes forever
- Just right → you steadily converge to a good solution

### Step 5: Repeat

Do this for every example in your dataset. One pass through the entire dataset is called an **epoch**. Most networks train for tens or hundreds of epochs.

---

## A Real Training Example

Here's what training a network to recognize handwritten digits (0-9) looks like:

```
Epoch  1: Loss = 2.31, Accuracy = 11% (random guessing among 10 digits)
Epoch  5: Loss = 0.45, Accuracy = 87%
Epoch 10: Loss = 0.15, Accuracy = 95%
Epoch 20: Loss = 0.05, Accuracy = 98%
```

The network starts with random weights (accuracy = random guessing). After 20 passes through the training data, it correctly identifies 98% of handwritten digits.

---

## Overfitting: When the Network Memorizes Instead of Learns

A network can get 100% accuracy on training data but fail on new data. This is called **overfitting** — it memorized the training examples instead of learning general patterns.

It's like a student who memorizes every answer in the textbook but can't solve a new problem.

**How to prevent it:**

| Technique | What it does |
|-----------|-------------|
| **More data** | Harder to memorize when there are millions of examples |
| **Dropout** | Randomly turns off neurons during training, forcing others to pick up the slack |
| **Early stopping** | Stop training when performance on test data starts getting worse |
| **Regularization** | Penalizes large weights, keeping the model simple |

---

## Types of Neural Networks

Different architectures are designed for different types of data.

![Feed-forward, CNN, and Transformer architectures](/assets/images/nn-types.svg)

| Type | Best for | How it works | Example uses |
|------|---------|-------------|-------------|
| **Feed-Forward (FFN)** | Tabular data, simple classification | Data flows in one direction through layers | Spam detection, price prediction |
| **CNN** (Convolutional) | Images, spatial data | Slides small filters across the image to detect patterns | Photo recognition, self-driving cars |
| **RNN / LSTM** | Sequential data | Has memory of previous inputs | Speech recognition, time series |
| **Transformer** | Text, sequences, almost everything | Every element attends to every other element | ChatGPT, translation, code generation |

**Transformers** have largely replaced RNNs for text because attention can process all words in parallel, while RNNs must go one word at a time.

---

## From Neural Networks to Deep Learning

**Deep learning** just means neural networks with many hidden layers (typically 10+). "Deep" refers to the depth of the network — more layers.

Why go deep? Each layer learns increasingly abstract features:

```
Layer 1:  edges, gradients, color blobs
Layer 2:  textures, corners, simple shapes
Layer 3:  eyes, ears, wheels, windows
Layer 4:  faces, cars, dogs, buildings
...
Layer 50: complex concepts, abstract reasoning
```

More layers = more abstraction = more powerful representations. This is why modern models like GPT-4 have 96+ layers.

---

## Common Terms You'll Hear

| Term | Plain English |
|------|--------------|
| **Parameters** | All the weights and biases in the network. "7 billion parameters" = 7 billion adjustable numbers |
| **Epoch** | One complete pass through all training data |
| **Batch** | A small chunk of training data processed at once (e.g., 32 images) |
| **Learning rate** | How big each weight update step is |
| **Loss** | A number measuring how wrong the network is |
| **Gradient** | The direction and steepness of the slope (tells you which way is "downhill") |
| **Inference** | Using a trained network to make predictions on new data |
| **Fine-tuning** | Taking a pre-trained network and training it a bit more on your specific data |

---

## Putting It All Together

A neural network is:

1. **Layers of simple math units** (neurons) connected by weights
2. **Trained by example**: show it data with correct answers, measure errors, adjust weights
3. **Learned, not programmed**: the patterns emerge from data, not from human rules

The entire magic boils down to two things: **a good architecture** and **a lot of data**. Everything else — the training loop, backpropagation, gradient descent — is just the machinery that turns data into useful weights.

When someone says "AI," most of the time they mean a neural network that was trained on a very large dataset. The concepts in this post are the foundation of all of it.
