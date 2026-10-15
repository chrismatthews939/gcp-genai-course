# Gen AI Strategy

## Top-down and bottom-up strategy

## AI notes


Augmentation versus automation
Use gen AI to enhance or augment your strategic thinking for:



Critical thinking and problem solving: Gen AI can provide data and insights, but humans are still needed to interpret those insights and make informed decisions.

Creativity and innovation: Gen AI can assist in generating ideas and exploring possibilities, but human ingenuity is still essential for pushing boundaries and developing truly innovative solutions.

Relationship building and collaboration: Gen AI can facilitate communication and information sharing, but strong interpersonal skills are still crucial for building trust, fostering collaboration, and navigating complex human dynamics.

Strategic planning and vision: Gen AI can help with forecasting and trend analysis, but human leadership is essential for setting a long-term vision, defining goals, and charting a course for the future.

Use gen AI to automate tasks that are:



Repetitive and rule-based: data entry, information retrieval, content formatting, and basic code generation.

Time-consuming and resource-intensive: research, data analysis, content summarization, and initial draft creation.

**Supervised machine learning** trains models on labeled data, where each input is paired with its correct output, allowing the model to learn the relationship between them. The model's goal is to identify patterns and relationships within this labeled data, enabling it to accurately predict outputs for new, unseen inputs.



Predicting housing prices is a common example of supervised learning. A model is trained on a dataset where each house has labeled data, such as its size, number of bedrooms, location, and the corresponding sale price. This labeled data allows the algorithm to learn the relationship between the features of a house and its price. Once trained, the model can then predict the price of a new house based on its features.



**Unsupervised ML models** deal with raw, unlabeled data to find natural groupings. Instead of learning from labeled data, it dives headfirst into a sea of unlabeled data. 



For example, an unsupervised learning algorithm could analyze customer purchase history from your company's database. It might uncover hidden segments of customers with similar buying habits, even though you never explicitly labeled those segments beforehand. This can be incredibly valuable for targeted marketing or product recommendations.



Think of it as exploratory analysis. Unsupervised learning helps you understand the underlying structure of your data and uncover insights that you might not have even known to look for.


## Agents

- Reasoning loop. Iterative process
- Tools
- Workflows

The reasoning loop often utilizes advanced prompt engineering frameworks to guide its decision-making process. These frameworks can include:

- Simple rule-based calculations
- Complex thought chains
- Machine learning algorithms
- Probabilistic reasoning techniques

AI Layers

- Infrastructure
- Models
- Agents
- Platforms. The platform layer provides the foundation for building and scaling your AI initiatives. 

### Platform Layer

Agent Platform is Google Cloud's unified machine learning (ML) platform designed to streamline the entire ML workflow. It provides the infrastructure, tools, and pre-trained models you need to build, deploy, and manage your ML and generative AI solutions.

Vertex AI = Streamlines the workflow to manage and deploy

Agent Platform supports open frameworks, allowing you to use your preferred tools and models without vendor lock-in.
Develop, deploy, and manage your models with ease using Agent Platform's integrated tools.
Leverage Google Cloud's scalable and reliable infrastructure to handle your ML workloads.
Fine-tune and adapt models to your specific needs using the built-in IDE.
Choose from a wide variety of pre-trained models or build your own.
Integrate your models into applications seamlessly using available APIs.

#### Agent Platform's MLOps tools

Agent Platform also includes ML operations (MLOps) tools built in to help you orchestrate end-to-end ML workflows, perform feature engineering, run experiments, manage and iterate your models, track ML metadata, and monitor and evaluate model quality. MLOps helps automate, standardize, and manage the ML project lifecycle, enabling better collaboration and continuous improvement.

#### Gemini Enterprise Agent
- Feature Store. Share, serve, and reuse ML features to maintain consistency and efficiency.
- Model Registry. Manage model versions, track changes, and organize your models throughout their lifecycle.
- Model Evaluation. Quickly evaluate and compare model performance to identify the best model for your use case.
- Workflow orchestration
- Model Monitoring. Monitor models for performance degradation, detect input skew and drift, and trigger updates or retraining.

### Model Layer

At the heart of every AI and machine learning system lies the model. Think of it as the brain of the operation. These aren't just any algorithms. They're sophisticated mathematical structures trained on massive amounts of data. This training process allows them to learn patterns and relationships, ultimately enabling them to perform a wide range of tasks, such as generating content, analyzing data, and classifying information.

Agent Platform: A model hub
Within Agent Platform, you have multiple options for how to handle AI models for your project. You can choose to build a model from scratch or use an existing model.

Use models with Model Garden

For many use cases, your organization won’t need to develop a new model. There are many AI models that already exist today that you can fine-tune and leverage for so many different use cases. Model Garden on Agent Platform is a service that lets you discover, customize, and deploy existing models from Google and Google partners.

Model Garden gives you options to pick from over 160 models and offers options of Google models, third-party models, and open-source models. With Agent Platform, you can customize these models with your own data and deploy them to your custom applications. Additionally, some pre-trained models can be used out-of-the-box without tuning.

#### First-party foundation models

Leverage state-of-the-art models from Google including Gemini models and task-specific models for image generation, speech-to-text, and more.

First-party pre-trained APIs
Build and deploy AI applications faster with our pre-trained APIs powered by the best Google AI research and technology.

Open models
Access a variety of enterprise-ready open models.

Third-party models
Model Garden supports third-party models from partners with foundation models.

Build models with Agent Platform



With Agent Platform, you have two options for training and using your own models. You can go fully custom and create and train models at scale using any ML framework (PyTorch, TensorFlow, scikit-learn, or XGBoost), or there are also options for using AutoML on Gemini Enterprise Agent Platform (Agent Platform AutoML) to create and train models with minimal technical knowledge and effort.



Agent Platform AutoML allows you to build these types of models:

Data type	Supported objectives
Image data	Classification, object detection.
Video data	Action recognition, classification, object tracking.
Tabular data	Classification/regression, forecasting.

Workflow
1. Gather your data
2. Prepare your data
3. Train
4. Manage your model
5. Deploy and predict

