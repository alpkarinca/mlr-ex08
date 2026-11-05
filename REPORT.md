# Report: a better leaf classifier (Task 4)

## What I did
I used a coding agent for the code and kept the code simple. Every variant is the Task 2 model (MobileNetV3-small, pretrained on ImageNet) with only a few options changed. I looked at the options and the results myself.

- **Data:** 101 training photos and 28 validation photos from our group (4 species, 96 x 96).
- **Same for every variant:** the same training images, 15 epochs (30 where written), Adam with learning rate 0.001, batch size 32.
- **3 seeds** (0, 1, 2) for every row. The table shows the mean ± the standard deviation over the 3 seeds.
- **Three test sets (accuracy in %):**
  - **Val:** our 28 validation photos.
  - **Camera-like:** the same 28 photos, each one 10 times randomly cropped, rotated (up to 30°) and with changed colours. I made this set once with a fixed seed (280 images). It is not really independent, because it comes from the same 28 photos.
  - **Last year:** 120 validation photos from last years' students. No variant was trained on these photos.

## Results

| Variant (one change vs. baseline) | Val | Camera-like | Last year |
|---|---|---|---|
| **Baseline** (Task 2 model, whole network trained) | 69.0 ± 7.3 | 63.1 ± 0.7 | 52.8 ± 3.5 |
| Only last layer trained (rest frozen) | 67.9 ± 2.9 | 69.4 ± 1.2 | 57.5 ± 1.4 |
| Geometric augmentation (crop, flips, rotation) | 75.0 ± 2.9 | 67.5 ± 2.0 | 61.4 ± 4.8 |
| Geometric + colour augmentation | 69.0 ± 1.7 | 65.8 ± 5.3 | 61.1 ± 2.7 |
| Cosine scheduler, 30 epochs | 67.9 ± 7.7 | 60.7 ± 1.8 | 51.1 ± 3.1 |
| Label smoothing 0.1 | 65.5 ± 6.1 | 60.0 ± 5.0 | 49.2 ± 5.4 |
| AdamW (weight decay 0.05) | 67.9 ± 5.1 | 61.2 ± 3.4 | 52.2 ± 4.4 |
| Dropout 0.5 (instead of 0.2) | 61.9 ± 3.4 | 57.3 ± 3.4 | 54.2 ± 5.4 |
| Keep the best epoch (chosen on val) | 73.8 ± 9.4 | 61.3 ± 3.5 | 55.0 ± 4.5 |
| Frozen + geometric + colour augmentation | 60.7 ± 5.1 | 71.0 ± 1.7 | 57.2 ± 1.0 |
| **Augmentation + cosine, 30 epochs** | 75.0 ± 5.8 | 71.3 ± 3.7 | **72.2 ± 2.8** |
| Augmentation + cosine 30 + label smoothing + AdamW | 75.0 ± 5.1 | 69.8 ± 1.9 | 68.9 ± 2.7 |

Rows with last years' photographs as **extra training data** (these runs were stopped early, so there are fewer seeds):

| Variant | Seeds | Val | Camera-like | Last year* |
|---|---|---|---|---|
| Baseline + last years' photos | 2 | 76.8 ± 5.4 | 64.1 ± 0.9 | 89.6 ± 1.3 |
| Augmentation + cosine 30 + smoothing + AdamW + last years' photos | 1 | 82.1 | 78.2 | 92.5 |

\* Here the last-year test photos come from the same year as the extra training photos (other students, but the same kind of photos). So these numbers are too good and cannot be compared with the other rows. One seed has no standard deviation.

## How big is the noise?
- One validation photo is worth 1/28 = **3.6 points**. The seeds differ by 2–9 points (the baseline has ± 7.3 on val). So differences smaller than about 5–7 points on val are **not reliable**.
- The camera-like set has 280 images, but only 28 different photos, so it is also noisy. The last-year set (120 photos from other students) is the best test for "new photos". Its noise is about ± 3–5 points.
- Only 3 seeds: the standard deviations are also rough.

## What each change brought
- **Geometric augmentation:** helped a little. Val +6, camera-like +4, last year +9. The val gain is about the same size as the noise, the last-year gain is a bit larger than the noise. I think it helps, but I am not 100 % sure.
- **Colour augmentation on top:** no further gain (val −6 vs. geometric only, last year the same). Within the noise, so no proof that it helps or hurts.
- **Only last layer trained:** not better on val, but better on camera-like (+6) and last year (+5), and the seeds differ less (± 1–3). Training only the last layer is stable.
- **Augmentation + cosine scheduler + 30 epochs:** the best of the complete rows. Val 75, camera-like 71, last year 72. The last-year gain (+19 points) is clearly bigger than the noise. The camera-like gain (+8) is bigger than the noise too. The val gain (+6) is not.
- **More epochs and cosine scheduler alone (no augmentation):** no gain. So the gain above comes from the augmentation together with the longer training.
- **Last years' photos as extra training data:** the val accuracy goes up (77–82), but the last-year result cannot be compared (see the note). On the camera-like set the baseline did not improve (64), the combination had 78 with only 1 seed. I would need more seeds to say more.

## What did not help
- Label smoothing, AdamW and dropout 0.5: all a bit worse or the same as the baseline, within the noise.
- Adding label smoothing and AdamW to "augmentation + cosine": no gain (last year 72 → 69).
- **Keeping the best epoch:** val looks better (73.8), but camera-like (61) and last year (55) did not improve. The reason: I chose the epoch with the validation photos, so the val number is too optimistic.
- Global average pooling: not tried. MobileNet already has global average pooling inside, so there is nothing to change.

## Which model for the phone
I would take **augmentation (geometric + colour) + cosine scheduler + 30 epochs**. It is the best complete row on camera-like and last year, and it needs only a few extra lines. Adding last years' photos looks promising, but I have only 1–2 seeds for it.

Not done yet: I have **not** exported this model and tried it in the course camera app. I must do this myself before I hand in.

## Limits
- Very small validation set (28 photos).
- The camera-like set comes from the same 28 photos.
- Runs with last years' photos were stopped early.
- The standard deviation is computed over 3 seeds only.
