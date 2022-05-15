# HumorDetectionBert_TFX
 
This repository houses a simple [BERT](https://github.com/google-research/bert) model trained for humor detection deployed using a [TensorFlow Extended](https://www.tensorflow.org/tfx) (TFX) end-to-end pipeline.  The model utilizes a [Kaggle dataset](https://www.kaggle.com/datasets/deepcontractor/200k-short-texts-for-humor-detection) consisting of 200k short texts and a binary target column determining whether or not those texts involve humor or not.  Labels have been encoded to be integers 0 (no humor) or 1 (humor).

## Content
- Pipeline [[link]](Pipeline/humor-detection-BERT.ipynb)[![Open In Collab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1zfFBTeC5zjBZ3fSz90GA25B5NIYzXBbF?usp=sharing)
- Data [[link]](Data/humor-detection.csv)

### Pipeline description
- The `ExampleGen` component will take our csv data and convert it into working examples for our other pipeline components. It will provide consistent splits and shuffles the dataset for ML best practice.
- The `StatisticsGen` component generates features statistics over both training and serving data provided by `ExampleGen`, which can be used by other pipeline components.
- The `SchemaGen` component creates a schema based on the statistics of our data (in this case provided by `StatisticsGen`). Schemas can specify data types for feature values, whether a feature has to be present in all examples, allowed value ranges, and other properties.
- The `ExampleValidator` component identifies any anomalies in the example data by comparing data statistics computed by the `StatisticsGen` pipeline component against the schema we created using `SchemaGen`. The inferred schema codifies properties which the input data is expected to satisfy.
- The `Transform` component performs feature engineering on our examples created by the `ExampleGen` component, using our data schema created by the `SchemaGen` component, and emits both a SavedModel as well as statistics on both pre-transform and post-transform data.
- The `Tuner` component tunes the hyperparameters for the model.
- The `Trainer` component will train our BERT model.
- `Resolver` is a special TFX node which handles special artifact resolution logics that will be used as inputs for downstream nodes.
- The `Evaluator` component performs deep analysis on the training results for our models, to help us understand how our model performs on subsets of the data. The `Evaluator` can also optionally validate exported models, ensuring that they are "good enough" to be pushed to production.
- The `Pusher` component is used to push a validated model to a deployment target during model training or re-training. Before the deployment, Pusher relies on one or more blessings from other validation components to decide whether to push the model or not. We'll use the results from our `Evaluator` component to determine whether or not our model is ready to be pushed.
