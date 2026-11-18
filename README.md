# A Simple Neural Network

An interactive teaching tool that shows a small fully connected neural network
learning to recognize handwritten digits, live in the browser.

**Live version:** https://tmillhouua.github.io/FullyConnectedNetworkViz/

The network reads a 14×14 image (196 pixels), passes it through two hidden
layers of 10 neurons each, and outputs a probability for each digit 0–9:

```
196 inputs → 10 hidden → 10 hidden → 10 logits → softmax → Best Guess
```

## Running it

Open `index.html` in a browser. There is no server, build step, or
dependency: the page is a single self-contained file, and the data loads from
`mnist14.js` through a plain `<script>` tag, which also works offline and when
the file is opened straight from disk.

## Reading the diagram

- **Input image (left):** the digit being classified. It also appears as the
  thin column of 196 squares, one per pixel.
- **Lines (weights):** blue is positive and red is negative. Thicker lines
  are larger weights.
- **Neurons:** a neuron's size shows its activation for the image on screen.
  Hidden neurons are scaled within each layer, so the most active neuron in a
  layer is drawn at full size.
- **Output column:** one neuron per digit, labelled 0–9. Its size shows that
  digit's *probability*, on a fixed scale from 0 to 1 (see below).
- **Best Guess:** the most likely digit and its probability.
- **Accuracy panel:** shows test accuracy (on 2,000 held-out images),
  training accuracy (on 2,000 training images), iterations, epochs, and the
  current learning rate.

## Controls

| Control | What it does |
|---|---|
| **Train / Pause** | Starts or stops training. Training covers about one pass over the 10,000 training images per second. |
| **Resample** | Shows a new input image. Before the first epoch ends, this is a random training image. After that, it steps through the digits in order (0, 1, 2, …) and shows one of three test images that best represent the network's typical response to that digit; these are updated each epoch. |
| **Reset** (click) | Pauses training and restores the weights from the last Initialize, so you can retrain from the same starting point. The training order is reshuffled, so the rerun won't follow exactly the same path. |
| **Reset** (hover, mouse only) | Temporarily shows the *initial* network for comparison: its weights, its neuron sizes, and its Best Guess for the current image. Move the mouse away to return to the trained network. |
| **Initialize** | Pauses training and starts over with new random weights (He initialization). |
| **Draw on the input image** | The cursor becomes a pen. Draw a digit (lift and press again to add strokes), then click anywhere else to finish. The drawing is converted the way MNIST was prepared: it is scaled to fit a 10×10 box, centred by centre of mass, and downsampled to 14×14. Its label shows as "?". |
| **Click the accuracy panel** | Shows the network's worst miss for each digit: the test image to which it gives the lowest probability of being its true digit. The caption reads "true → guess". The worst miss may be a correct guess made with low confidence. Click a thumbnail to load it as the input. |

### Neuron buttons

Clicking any neuron selects it and shows a heatmap below the Best Guess box,
with three modes. Click the background to close the heatmap.

| Mode | What the heatmap shows |
|---|---|
| **Weights** | The neuron's incoming weights. A first-layer neuron's 196 weights form a 14×14 image; later neurons' 10 weights form a column. Blue is positive and red is negative. |
| **Favorites** | The average training image, weighted by how much each image activates the neuron: roughly, *what turns this neuron on*. Shown in grayscale. |
| **Contribution** | For each pixel, how much ink there pushes the neuron's weighted sum up (blue) or down (red), on average. Each pixel's value is multiplied by the gradient with respect to that pixel. |

Favorites and Contribution use 2,000 training images and refresh while
training. Each heatmap is scaled to its own largest value. For output
neurons, the heatmap label reads "Output for *j*".

## Choices to note: the output (logit) column

The output column is a **display of the classifier's existing logits**, not an
extra layer of the network. In the code, the final 10×10 weight matrix and
softmax belong to the classifier; the diagram draws its weights as a third
layer of lines and its 10 logits as a third column of neurons.

- **Sized by probability, not by logit.** Raw logits are unbounded and shift
  together, so sizing by logit (even after subtracting the minimum) made many
  output neurons look large when the network was confident in one digit.
  Sizing by softmax probability on a fixed 0–1 scale means one large neuron
  reflects one confident guess. An untrained network shows ten roughly equal,
  smallish neurons (about 10% each).
- **Same colours as the other neurons.** The output neurons use the same
  colour gradient as the hidden neurons, with a digit label on each.
- **Plain lines into Best Guess.** The lines from the output neurons to the
  Best Guess box are uniform gray. They don't encode weights, because softmax
  has none.
- **Heatmaps work for outputs too.** Weights shows the output neuron's 10
  incoming weights. Favorites and Contribution are computed on the logit, the
  value before softmax, which has no ReLU.

## Training details

- **Activation:** Leaky ReLU (slope 0.1 for negative inputs), so neurons
  don't "die".
- **Loss and output:** softmax with cross-entropy.
- **Optimizer:** mini-batch SGD with batch size 32.
- **Learning rate:** starts at 0.05 and halves every 10 epochs, at most three
  times (to 0.00625). It resets on Initialize and Reset.
- **Regularization:** an L1 penalty on all weights (`L1_PENALTY` in
  `index.html`) pushes unhelpful weights to exactly zero.
- **Initialization:** He initialization for the hidden layers; standard
  deviation √(1/n) for the classifier.

## Files

- `index.html`: the entire app: network, training, and visualization. Its
  settings are constants near the top of the script and are commented.
- `mnist14.js`: the data, generated; do not edit it.
  - 10,000 training and 2,000 test images, class-balanced.
  - Downsampled from 28×28 to 14×14.
- `tools/build-data.js`: regenerates `mnist14.js` from the original MNIST
  files. Run `node tools/build-data.js` (no packages needed).

## Data credit

MNIST is by Yann LeCun, Corinna Cortes and Christopher J.C. Burges. It is
commonly distributed under CC BY-SA 3.0.
