---
title: "RoBERTa Model for Merchant Categorization at Square"
datePublished: Mon Mar 10 2025 05:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm8q3lhpp000i0ajpfc5jh1hx
slug: roberta-model-for-merchant-categorization-at-square
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1743004030346/c744354f-2655-41cf-92f6-f4287153a418.png

---


Note: In this blog post, the words *merchant* and *seller* will be used interchangeably.

# Introduction

Square’s vast ecosystem of products enables businesses to accept payments, receive banking services, utilize payroll, schedule appointments, and more. With its mission of *economic empowerment*, Square’s growth is grounded in enabling businesses to further scale their operations over time. In order to achieve this, Square’s overall product development, marketing strategies, compliance considerations, financial forecasting, and more rely heavily on a deep understanding of the types of merchants that it serves. Indeed, the company has long recognized the importance of *merchant categorization*, and over the years has made many attempts to tackle this challenging problem. 

In short, merchant categorization refers to the process of determining the type of business that a seller is running. Numerous techniques for achieving this exist, ranging from merchant self-selection to manual review to machine learning (ML), or a combination thereof. At Square, it's historically been challenging to develop an ML solution that met our business needs, leading to inefficiencies and missed opportunities. This blog post summarizes our latest efforts at achieving significantly more accurate merchant categorization, delving deeply into the following topics: 

* Why accurate categorization of merchants is important for Square
* The end-to-end development of an ML model, based on the RoBERTa (Robustly Optimized BERT Pretraining Approach) architecture, that achieved a significant leap forward in merchant categorization accuracy
* The business impact of this large effort, as well as next steps

## Why is categorizing merchants correctly important?

As mentioned previously, given the seller-centric nature of Square’s business, numerous internal initiatives rely heavily on insights into the types of merchants that we serve. Broadly speaking, Square’s merchant base can be categorized into 4 themes, which we will call “audiences”:

* Food and Drink
* Health and Beauty
* Retail
* Services

Within each audience, additional granular categories (internally defined by Square) also exist. It should not be difficult to imagine that business strategy, organizational structure, metrics tracking, and overall prioritization must take merchant categorization into account. We provide several examples below:

1. **Personalized Product Experiences**: Square can tailor particular products so that they better represent the type of customer that a merchant primarily serves. For instance, on webpages and within apps, we can call customers “clients” for sellers in the beauty space, but “customers” for sellers in the retail space.
2. **Business Insights and Strategy**: Accurate merchant categorization provides valuable data for business insights and strategic decision-making. Investing heavily in product and engineering for sellers who we think are massage therapists, for example, could be misguided if many are actually retail merchants selling massage products, leading to poor business decisions.
3. **Growth Strategy**: Correct categorization allows us to send targeted marketing communications and product recommendations to merchants that are relevant to their business.
4. **Product Eligibility**: Merchant categorization can help determine eligibility for various products and services. For example, if we were beta launching a product for sellers in the food and drink space, then we’d want to make sure the maximum number of eligible merchants can be reached.

Beyond the aforementioned use cases, accurate merchant categorization can also have impact on Square’s interactions with card issuers:

Note: *Payment processors and card issuers require accurate Merchant Category Codes (MCCs) as part of the payment activation process. Square delivers this information to them during transactions.*

1. **Interchange Fees**: MCCs are a factor in determining interchange fees that Square pays for processing card transactions. This is because each MCC carries a unique risk profile with varying levels of fraud, chargebacks, and regulatory scrutiny. For example, a luxury good shop likely has a higher chance of chargebacks compared to a barber shop - and this is reflected in the fee structure. Accurate categorization ensures that Square is charged the correct fees and not overpaying unnecessarily.
2. **Rewards and Incentives**: Card issuers often provide rewards and incentives based on the MCC. Cards that provide cashback for dining out, for instance, need to know if a merchant is a restaurant. 

# The Challenges of Merchant Category Self Selection

Historically, Square has relied heavily on self-selection during new merchant onboarding as the method to determine a merchant’s business category. This means that sellers are asked to choose their business category from a pre-defined list when they first sign up for Square. As an illustrative example, under the Beauty category, there are numerous subcategories, such as “Hair Salon” and “Barber Shop”.

Via extensive manual review of individual sellers, we noticed that merchant self-selection is highly susceptible to miscategorization. This is due to several reasons:

1. **Speed Over Accuracy**: Merchants often want to complete the onboarding process quickly and may not spend enough time selecting the most accurate category.
2. **Granular Categories**: For example, a business offering both nail and hair services might struggle to choose between "Nail Salon" and "Hair Salon".
3. **Lack of Clear Definitions**: Many merchants are unsure which category best fits their business because Square does not explicitly indicate category definitions in its product flows. For instance, if “design” were a potential option from which to choose, then both hairdressers and web designers could potentially self-identify this way, both unaware of how Square interprets this particular nomenclature.

## Historical ML Approaches to Merchant Categorization at Square

While there have been efforts across Square to improve upon merchant categorization accuracy, most have suffered from several key issues:

* They were built using relatively older ML methods
* They often over-indexed on self-selected information as the source of truth for labels;  as discussed previously, self-selection is highly inaccurate and misrepresents a good portion of Square’s seller base
* They often narrowly focused on new seller onboarding use cases, and did not make use of key post-onboarding information

# The Solution: RoBERTa Merchant Categorization Model

![image3](//images.ctfassets.net/1wryd5vd9xez/5t1mxLJOOSTbkfypkBuqod/bd0f4b9dd96a161f4a99fbe87abbe16c/image3.png)

<p align="center" style="font-size: 14px; font-style: italic"><strong>Figure 1</strong>: High-level Model Workflow Overview</p>

To address the aforementioned issues, we leveraged 3 key ingredients in the ML workflow:

1. **Quality Training Data:** The model is trained on a random sample of over 20,000 manually reviewed sellers, ensuring high-quality ground truth labels for our training and evaluation data. 
2. **LLM Architecture:** By utilizing the RoBERTa architecture, we harness the power of LLMs to achieve superior categorization accuracy.
3. **Post-Onboarding Predictive Signals:** Our approach uses a wide array of post-onboarding signals, including business names, self-selected onboarding information, and the entire catalog of items and/or services offered by the sellers, ranked by frequency of purchase.

By combining these diverse inputs, the RoBERTa model can make highly accurate predictions about a merchant's category, significantly improving upon previous methods. We will now walk through these steps in greater detail (see Figure 1 for a high-level summary of the end-to-end workflow).

### Data Preprocessing

**1. Remove Auto-Created Services**

At Square, we auto-create items and services for merchants based on their self-selected business in order to help them get “up and running” more quickly and to more easily understand (in product) how to view and update catalog items. However, this also creates additional noise: if a merchant accidentally selects the wrong category info, it will receive auto-created services that don’t represent their business. For instance, a seller named “Jay’s Real Estate” may end up with services like “Shampoo, Conditioner, Haircut” because they selected “Hair Salon” as their business category. As part of data preprocessing,  we remove all auto-created services that were not later manually modified by the seller (taking the view that services that were altered are typically ones where the merchant actually wants to keep them).

**2. Rank by Purchase Frequency**

Many ML models have a “token” (essentially a word) limit that they can process. Some merchants sell *a lot* of things (e.g. possibly tens of thousands), and so we need to truncate their catalog items to a level that is below the token amount that the chosen ML model architecture can process. In order to do this, for each merchant, we rank their catalog by the amount of times that item was purchased, and subsequently cut off any items beyond the token threshold. This ensures the model is categorizing a merchant based on what they sell *the most*, which should also be more indicative of what they actually do.

**3. Prompt Formatting**

Akin to prompt engineering, which is the process of creating and refining natural language instructions to guide generative AI models to produce desired outputs, we also carry out analogous preprocessing to create clearer *context*. To illustrate the above, for the sample raw data in Table 1,  the model code would generate a single text called the “Input String”.

<table>
  <tr>
   <td><strong>Business Name</strong></td>
    <td>Massage Flow
   </td>
  </tr>
  <tr>
   <td><strong>Service Names Ranked</strong></td>
    <td>Full Body Massage, Intense Hydration Massage, Hot Stone Massage, Deep Tissue Massage (Auto Created and Edited), Swedish Massage (Auto Created)
   </td>
  </tr>
  <tr>
   <td><strong>Activation Business Category</strong></td>
    <td>health_care_and_fitness
   </td>
  </tr>
  <tr>
   <td><strong>Activation Business Subcategory</strong></td>
    <td>massage_therapist
   </td>
  </tr>
   <tr>
   <td><strong>Input String</strong></td>
    <td>Business Name: Massage Flow | Item Names: Full Body Massage, Intense Hydration Massage, Hot Stone Massage, Deep Tissue Massage | Onboarding Business Category: health care and fitness | Onboarding Business Subcategory: massage therapist
   </td>
  </tr>
  <tr>
   <td><strong>Human Reviewed Label</strong></td>
    <td>massage_therapist
   </td>
  </tr>
</table>

<p align="center" style="font-size: 14px; font-style: italic"><strong>Table 1</strong>: Sample Raw Data</p>

## Model Creation

Equipped with manually reviewed labels and the aforementioned model features, training was carried out via Databricks, in order to leverage the scalability offered by GPU-based clusters. Given the RoBERTa model is large, GPUs were needed to accelerate the training and tuning process.

Databricks allows one to create computer clusters with the desired machine type. Hence we adopted the below configuration with 1 A10G Nvidia GPU:

![image1](//images.ctfassets.net/1wryd5vd9xez/2Px9Hpn0kJZ1lJeNzt4vZg/b30e2e134135e2c5cc9eab8a55eb6866/image2.png)

Prior to model development with the aforementioned data, we first converted our human reviewed categories to numerical values:

```python
# Numerically encode data
from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()
category_encodings = le.fit_transform(df_new_training_data[human_reviewed_label])
df['category_encoding'] = category_encodings
num_categories = len(set(category_encodings))

# Create label2id mapping
label2id = {label: int(idx) for label, idx  in zip(le.classes_, le.transform(le.classes_))}
# Create id2label mapping
id2label = {idx: label for label, idx in label2id.items()}
```

In addition, as is common for text-based modeling, we tokenized our data so that it can be read by the LLM. *Tokenization* involves splitting input text into smaller units (tokens) so that a model family such as RoBERTa can process it:

```python
# Tokenize training data for model
from transformers import AutoTokenizer
from datasets import Dataset
model_name = 'roberta-large'
tokenizer = AutoTokenizer.from_pretrained(model_name)
def preprocess_function(X):
   return tokenizer(X['text'], truncation=True)   

train_dataset = Dataset.from_dict(train_data)
train_encodings = train_dataset.map(preprocess_function, batched=True)
```

From there, we instantiated our model using the AutoModelForSequenceClassification class from Huggingface and then loaded it onto our GPU. When you call from_pretrained, it initializes the model with weights that have been pre-trained on a large corpus of text which we will further train using our specific data.

```python
# Load model onto GPU
from transformers import AutoModelForSequenceClassification
device = 0 if torch.cuda.is_available() else -1
awt_model = AutoModelForSequenceClassification.from_pretrained(model_name, num_labels=num_categories, id2label=id2label, label2id=label2id).to(device)
```

Finally, we trained our model with its associated hyperparameters using the Trainer class (also from Huggingface). Note that some models are so large that they do not fit onto the GPU during training, leading to out of memory errors. In order to combat this, and to reduce overall memory usage, the parameters FP16 and Gradient Checkpointing were utilized.

**FP16** refers to using 16-bit floating-point numbers instead of the standard 32-bit (FP32) for computations. **Gradient checkpointing** stores fewer intermediate activations - instead of retaining all activations during a model pass, it recomputes them only during the backward pass.

```python
# Trainer args determined from research
from transformers import TrainingArguments, Trainer
learning_rate = 1e-05
warmup = .05
lr_scheduler_type = 'linear'
epochs = 4
evaluation_strategy = 'no'
save_strategy = evaluation_strategy
batch_size = 16

training_args = TrainingArguments(
   output_dir=training_output_dir,
   optim = 'adamw_torch',
   learning_rate=learning_rate,
   per_device_train_batch_size=batch_size,
   per_device_eval_batch_size=batch_size,
   num_train_epochs=epochs,
   weight_decay=0.01,
   evaluation_strategy=evaluation_strategy,
   save_strategy=save_strategy,
   load_best_model_at_end=False,
   push_to_hub=False,
   save_total_limit=1,       
   lr_scheduler_type = lr_scheduler_type,
   warmup_ratio = warmup,
   disable_tqdm = False,
   fp16=True,
   gradient_checkpointing=True       
)

trainer = Trainer(
   model=awt_model,
   args=training_args,
   train_dataset=train_encodings,
   tokenizer=tokenizer,
   data_collator=data_collator,
   compute_metrics=compute_metrics
)

trainer.train()
```

## Inference

This model performed inference daily and, given the vast number of sellers on the platform (in the tens of millions!), was another area that demanded even more GPU support. To generate predictions for all of these merchants without weeks of delay, we leveraged 4 different techniques:

1. **Multiple Workers:** Recall that for training the model we utilized 1 A10G Nvidia GPU. For inference, we increased the “Max Workers” number which determines the amount of workers that are in the compute cluster (each worker in this case has 1 GPU). This allowed us to utilize multiple GPUs in parallel to predict more merchants at once.
2. **PySpark:** PySpark excels at parallel processing by efficiently distributing computations across multiple GPUs or nodes in a cluster. This allows us to handle significantly larger datasets and perform complex computations at scale.
3. **Batch Size Optimization:** Batch size is a parameter that determines how many samples (e.g., text sequences) are processed together during inference. Larger batch sizes can improve efficiency by making better use of GPU resources, however this benefit is only realized up to a certain point, as excessively large batch sizes can cause memory issues. To find the optimal batch size for our use case, we tested various ones on sample data.
4. **Incremental Predictions:** We avoided running predictions for sellers whose inputs remained unchanged from the previous day. For example, if a seller’s business name, services, and self-selected activation info had not been updated, we retained their existing business category prediction without rerunning inference. This approach significantly reduced inference time by focusing only on sellers with updated inputs.

```python
from transformers import pipeline
device = 0 if torch.cuda.is_available() else -1
batch_size = 8
classifier = pipeline('text-classification', model=model_path, truncation=True, padding=True, device = device, batch_size = batch_size)

# Sample PySpark function
@pandas_udf('string')
def return_top_predictions_udf(texts: pd.Series) -> pd.Series:
 predictions = [prediction['label'] for prediction in classifier(texts.to_list())]
 return pd.Series(predictions)

df = df.withColumn(top_sub_category_classifier_output, return_top_predictions_scores_udf(F.col(combined_processed_col)))
```

![image1](//images.ctfassets.net/1wryd5vd9xez/1BXrvDYoIOa6NW860pttIm/1b6e8fde03f2eaf41c657b7371d59df7/image1.png)

The model produced two key output tables to serve distinct purposes:

1. **Historical Table**: Stored daily partitions of predictions, enabling troubleshooting and monitoring over time for things such as model drift, validating whether the model’s performance remained consistent.
2. **Latest Predictions Table**: Contains the most up-to-date predictions, readily available for stakeholders to use in decision-making (e.g. powering key business metrics). 

## Model Evaluation

The model is estimated to have an absolute ~30% improvement in accuracy of determining seller business category than existing methods.

By examining model performance on our test data (randomly selected samples that were held-out and not part of our training data), we summarize the accuracy gains across a few major business categories, relative to merchant self selection, in Table 2:

<table style="margin: auto;">
  <tr>
   <td><em>Human Reviewed Category</em>
   </td>
    <td><p>Raw % Accuracy Increase</p>
   </td>
  </tr>
  <tr>
   <td>beauty_and_personal_care
   </td>
    <td><div style="text-align:center;">32%</div>
   </td>
  </tr>
  <tr>
   <td>food_and_drink
   </td>
   <td><div style="text-align:center;">13%</div>
   </td>
  </tr>
  <tr>
   <td>health_care_and_fitness
   </td>
   <td><div style="text-align:center;">22%</div>
   </td>
  </tr>
  <tr>
   <td>home_and_repair
   </td>
   <td><div style="text-align:center;">35%</div>
   </td>
  </tr>
  <tr>
   <td>leisure_and_entertainment
   </td>
   <td><div style="text-align:center;">18%</div>
   </td>
  </tr>
  <tr>
   <td>professional_services
   </td>
   <td><div style="text-align:center;">17%</div>
   </td>
  </tr>
  <tr>
   <td>retail
   </td>
   <td><div style="text-align:center;">38%</div>
   </td>
  </tr>
</table>

<p align="center" style="font-size: 14px; font-style: italic"><strong>Table 2</strong>: Accuracy gains achieved by the ML model

## Business Impact and Next Steps

The model output, which refreshes on a daily basis, is now utilized by Square to power all business metrics that require segmentation by merchant categories/audiences. As mentioned previously, this has important implications for product strategy, GTM strategy + targeting (including email campaigns), as well as numerous additional applications. In future work, we will describe the model’s further use cases for reducing interchange fees associated with payment processing (to which we gave a brief preview earlier in this blog post). 

## Co-Author

Rob Wang

## Acknowledgements

Thanks to Willem Avé and many other colleagues for technical discussions, manual review partnerships, and overall support.