# Oral presentation — speaking script

**ST2AIM Project — Teaching Computers to See**
Victor Haegeman · 10 minutes · 10 slides

> Each slide below carries the same text as the speaker notes inside the `.pptx`
> (PowerPoint → View → Notes Page, or the Notes pane under the slide).
> Timings are cumulative. Aim to finish slide 9 at 9:00 and leave the last minute for the conclusion.

---

## Slide 1 — Title · 0:00 → 0:30

Good morning. I built a convolutional neural network that tells cats from dogs, trained it on
2,000 images, and then tested two different ways of fixing the problem it ran into.

I'll go through the data, the architecture, the training, the results, and the two improvements.

*(Don't read the numbers on the slide out loud — they are there so the audience sees the scale while you talk.)*

---

## Slide 2 — The data · 0:30 → 1:45

Three thousand images, perfectly balanced between cats and dogs — so a coin flip scores fifty
percent, and that's the baseline every model has to beat.

The photos all come in different resolutions, so everything is resized to 150 by 150 in colour.

As the brief asks, I split the training folder 80/20: 1,600 images to train on, and 400 kept aside
as a test set the model never sees. The validation folder stays separate and lets me watch
generalisation at every epoch.

One thing worth looking at, bottom left: these are real photos. Free poses, occlusions, humans in
the frame, all sorts of lighting. This is much harder than a benchmark like MNIST, where the object
is centred and normalised.

---

## Slide 3 — Why a CNN · 1:45 → 3:00

Why convolution at all, rather than a plain dense network?

Flatten one of these images and you get 67,500 numbers. A single dense layer of a thousand neurons
is already 67 million parameters — and on top of that, flattening throws away which pixel was next
to which.

A CNN solves both problems with two ideas. **Local connectivity**: each neuron looks at a 3 by 3
window instead of the whole image. **Weight sharing**: the same kernel slides over every position,
so a pattern learned in one corner is detected everywhere, and the parameter count no longer
depends on the image size.

The table at the bottom is the four blocks I use. Convolution learns the filters. ReLU adds the
non-linearity — without it, stacking layers would collapse into a single linear map. Pooling shrinks
the feature map and has no weights at all. And the dense head makes the final decision.

---

## Slide 4 — Architecture · 3:00 → 4:15

Here's the model: four blocks, each one convolution, ReLU, pooling — exactly what the brief
specifies. The number of filters goes up, 32 to 64 to 128, while the resolution comes down. The
representation loses detail and gains meaning.

I recomputed the parameter count by hand, using the formula from the course — filters times kernel
size times input channels, plus one bias each. I get **1,043,905**, which is exactly what
`model.summary()` reports.

And there's something interesting in that number. Seventy-seven percent of the parameters sit in
one single dense layer — 802,944 out of a million. The four convolutional blocks together account
for only 240,000. That's the weight-sharing argument from the previous slide, in numbers.

---

## Slide 5 — Training · 4:15 → 5:30

Fifteen epochs, batches of 100 images as required. With 1,600 training images that's 16 batches per
epoch, so 240 weight updates in the whole run.

And the curves tell the story. Training accuracy climbs to 0.875. Validation flattens out around
0.70 from about the eighth epoch.

But the clearest signal is the loss, on the right. Validation loss bottoms out at epoch 13 at 0.585,
then climbs back to 0.743 — while the training loss keeps going down. That divergence is the
textbook signature of **overfitting**: the model keeps improving on data it has already seen, at the
expense of data it hasn't.

It's not underfitting — if it were, both curves would be low and close together. Here training
accuracy goes high, so capacity isn't the limit.

Three reasons: only 1,600 images, roughly 650 parameters per training image, and no regularisation
of any kind.

---

## Slide 6 — Evaluation · 5:30 → 6:45

On the 400 held-out test images: **72.8 percent accuracy**. Clearly above chance, so the model has
learned genuine visual features.

But the interesting part is precision versus recall. Precision is 0.84 — when the model says "dog",
it's right 84 percent of the time. Recall is only 0.62 — it finds just 62 percent of the actual dogs.

Read that in the confusion matrix: 83 dogs classified as cats, against only 26 cats classified as
dogs. The model is **conservative about the dog class** — at the slightest doubt, it answers "cat".

And since the test set is balanced, that's not a data artefact, it's where the decision boundary
ended up. Reporting accuracy alone would have hidden it completely.

---

## Slide 7 — Fix 1: data augmentation · 6:45 → 8:00

First fix: data augmentation.

On the left, the same photo eight times — flipped, rotated, zoomed. The transformations preserve the
label: a mirrored dog is still a dog. The model never sees exactly the same image twice, so it can't
memorise them.

The brief insists that augmentation must not touch the test set. Implementing it as Keras
preprocessing layers guarantees that: they're active during `fit` and automatically inactive during
`predict`. The evaluation protocol is unchanged.

The results are at the bottom. The train-validation gap goes from 0.179 down to **0.009** — the two
curves are practically on top of each other. Test accuracy rises to 0.750. And recall on dogs jumps
from 0.62 to 0.86, so the conservative bias from the previous slide is gone.

One caveat worth stating: augmentation does not create extra images. Every epoch still has 1,600
images and 16 batches. What changes is that they're transformed differently each time.

---

## Slide 8 — Fix 2: dropout + batch normalization · 8:00 → 9:00

Second fix, tested separately as the brief requires.

Dropout switches off half the units of the dense layer at every training step, so the network can't
rely on any single unit and has to spread the information out. Batch normalization normalises each
convolution output over the batch, which stabilises training and speeds it up.

This is the best of the three models: **0.780 accuracy**, F1 of 0.810, and the gap cut in half
compared with the baseline.

And one bug worth reporting. With Keras's default batch-norm momentum of 0.99, and only 240 weight
updates in the entire run, the running statistics it uses at inference never converge. My validation
accuracy was pinned at **exactly 0.500** for fifteen epochs while training looked perfectly healthy.
Setting the momentum to 0.9 fixed it completely.

---

## Slide 9 — Comparison · 9:00 → 9:40

Everything side by side.

Model C wins on score. Model B wins on the gap. And that's the point I'd make: the two techniques
attack the same problem from opposite sides — augmentation changes the **data**, dropout and batch
norm change the **network**. They're complementary, not competing, and combining them would be the
obvious next step.

As a bonus I set up transfer learning: a frozen MobileNetV2 backbone pre-trained on ImageNet, with a
new classification head. That's about 1,300 trainable parameters instead of a million.

---

## Slide 10 — Conclusion · 9:40 → 10:00

Three things I take away.

The limiting factor was the **data, not the architecture** — neither fix added a single filter, and
both improved generalisation.

Regularisation doesn't make a model smarter; it **removes its option to memorise**.

And you have to read the curves, not just the final number — one hyperparameter froze my validation
at exactly 0.500 while the training run looked fine.

Thank you — happy to take questions.

---

# Likely questions, and answers

**"Why 150 × 150 and not the original resolution?"**
A network needs a fixed input size, and the images all differ. 150 is a compromise: large enough to
keep the shapes that distinguish the two classes, small enough to train on CPU in a reasonable time.
Higher resolution would mean a bigger flattened volume and therefore an even heavier dense layer.

**"Why binary cross-entropy and not MSE?"**
Two classes, one sigmoid output, so binary cross-entropy is the negative log-likelihood of the
matching Bernoulli model. Its gradient with respect to the pre-activation is simply ŷ − y, so a
confident wrong prediction produces a large gradient. MSE combined with a sigmoid saturates and
learns much more slowly.

**"Why Adam rather than SGD?"**
Adam adapts the step size per parameter from running estimates of the gradient moments. It converges
faster here and is far less sensitive to the initial learning rate, which matters because I'm not
tuning a schedule.

**"Why did you stop at 15 epochs?"**
The brief allows 5 to 15. Beyond that the baseline would only overfit further — its validation loss
already turns upward at epoch 13. With `EarlyStopping` on the validation loss I'd have stopped
around there automatically.

**"Why is the test set taken from the train folder rather than the validation folder?"**
Because the brief asks for an 80/20 split of the training data. It also keeps the roles clean: the
validation folder is used to monitor every epoch, so it influences my choices; the test set is
touched exactly once, at the end, which is what makes it an honest estimate.

**"Could you have combined the two improvements?"**
Yes, and it would probably be better still. The brief asks to apply each technique separately, so
the individual contribution of each can be measured — combining them straight away would have made
the comparison impossible.

**"Why is your accuracy only 78% when this problem is considered solved?"**
The published results use the full 25,000-image Kaggle dataset, or transfer learning. With 1,600
images trained from scratch, 78% is in the expected range. That's exactly what the bonus shows: a
pre-trained backbone reaches far higher because it brings 1.4 million images' worth of learned
features with it.

**"What does the 0.5 threshold do, and would changing it help?"**
The sigmoid outputs a probability; 0.5 turns it into a class. Lowering it would raise recall on dogs
and lower precision. Since the classes are balanced and neither error is more costly here, 0.5 is
the neutral choice — but a precision-recall curve would be the right tool to justify any other value.
