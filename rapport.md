# Labo 3 - ARN
- Group : Bleuer Rémy, Duruz Florian
- Class : B
- Date  : 20.03.2026

## First experience

### Pretreatment
We chose the 25 lowest frequencies going from 1Hz to 25Hz, they cover all EEG bands that are relevent for sleep calssification. Frequencies that are higher carry only little discriminative infos for this task.

Using the `StandardScaler` centers each of the 25 features to mean 0 and std 1, thus stabilizing and speeding up training.

### Architecture
![Architecture](./assets/1st_experiment/model_summary.png)

We only use one hidden layer with 16 neurones as we only have 25 inputs to treat. Adding hidden layers and neurones would risk overfitting without having any benefits. The sigmoid output produces a probability between 0 and 1, and the threshold of 0.5 determines the predicted class. We also added a momentum of 0.9 so we avoid getting stuck in a local minima and speed up convergeance. And finaly, the learning rate of 0.01 is pretty standard, but we don't want it to be too high or too low as it could cause losses to the convergeance and slow down the training.

### Training history
![Training history](./assets/1st_experiment/3f_cv.png)

Both `train_loss` and `val_loss` drop pretty drastically ine the first two epochs and the decrease slowly and steadily at the same time. They converge around 0.084. There is no visible overfitting as the two curves remaing close during the training and `val_loss` never increases.

### Performance
![Global confusion matrix](./assets/1st_experiment/global_cm.png)

The accuracy is as follows : (13626 + 22665) / 40863 = 88.8%. With this result, we can affirm that the model correctly classifies the large majority of samples in both classes. The error that is mostly present is when the model predicts `awake` when the mouse is actually `asleep` (2877 cases), it is not completly absurd as the light n-rem has EEG patterns that are similar to `awake` state.

## Second experiment

### Pretreatment
Same as the first experiment, we chose the 25 lowest frequencies from 1Hz to 25Hz. The `StandardScaler` centers each feature to mean 0 and std 1. We also removed rows with missing values using `dropna()` to avoid NaN propagation during training.

### Architecture
![Architecture](./assets/2nd_experiment/2nd_model_summary.png)

Compared to the first experiment, the main changes are:
- **3 output neurons** instead of 1, one per class (rem, n-rem, awake)
- **Softmax** activation instead of sigmoid, each output represents a probability and the three sum to 1
- **Categorical crossentropy** loss instead of MSE, which is more appropriate for multi-class classification

The predicted class is determined by `argmax` of the output vector, which avoids the ambiguity of a threshold (with sigmoid, multiple classes or no class could be predicted simultaneously).

We kept the same single hidden layer with 8 neurons, the same learning rate of 0.001 and momentum of 0.99.

### Training history
![Training history](./assets/2nd_experiment/2nd_3f_cv.png)

Both `train_loss` and `val_loss` decrease steadily across the 50 epochs. The two curves remain close throughout training with no visible divergence, indicating no significant overfitting. The model converges smoothly, suggesting that the learning rate and momentum are well suited for this task.

### Performance
![Global confusion matrix](./assets/2nd_experiment/2nd_global_cm.png)

The model performs well on the **rem** and **awake** classes, which are the most represented in the dataset. However, **n-rem** is the most confused class, it is frequently misclassified as either rem (340 cases) or awake (1407 cases). This is expected as n-rem shares EEG characteristics with both other states, especially light n-rem which resembles the awake state.

| Fold | F1 rem | F1 n-rem | F1 awake | F1 micro |
|------|--------|----------|----------|----------|
| 1    | 0.886  | 0.449    | 0.943    | 0.912    |
| 2    | 0.887  | 0.483    | 0.942    | 0.911    |
| 3    | -      | -        | -        | -        |

The micro F1 score around **0.91** confirms that the model generalizes well overall, despite the difficulty of classifying n-rem.
