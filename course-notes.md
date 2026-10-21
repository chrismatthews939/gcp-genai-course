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

### The infrastructure layer
The foundation of gen AI solutions

The "infrastructure layer" is the foundation upon which any AI system is built. It's the combination of hardware and software that provides the necessary computing power, storage, and networking capabilities to train, deploy, and scale AI models. This layer is crucial for handling the massive datasets and complex computations involved in modern AI, especially with the rise of large language models and generative AI.

Google has invested heavily in building a robust and cutting-edge AI infrastructure, designed to handle the most demanding AI workloads. Here are some key components.

Imagine your company wants to undertake a new, bold gen AI project. However, the engineering team hits a major roadblock. Training the complex model requires significant processing power, far beyond the capabilities of your company's in-house servers. It takes hours, if not days, to train your specialized models and the long training times have started to hinder the team's progress to the point where it's delaying the product launch. Your existing infrastructure simply can't keep up, and the cost of scaling their own hardware is prohibitive. This is where high-performance computing, the heart of Google's AI infrastructure, comes in.

GPUs and TPUs
These specialized processors are the workhorses of AI. They excel at parallel processing, or doing multiple things at once, which is crucial for training large neural networks quickly and efficiently. GPUs were originally designed for graphics rendering, but since they offer general-purpose parallel processing power they’ve increasingly been used in the AI domain. TPUs are Google's custom-designed chips specifically optimized for AI tasks.

Hypercomputers
A hypercomputer is essentially a supercomputer built by connecting many individual computers (nodes) together. These nodes contain GPUs and TPUs. By linking them with high-speed networks, they work together as one massive computing unit. They provide the massive scale necessary for training and running the most demanding generative AI models.

Large-scale storage systems
Google Cloud's storage infrastructure is optimized for AI workloads, offering high throughput (the amount of work done within a specific period of time), scalability, and the ability to create dense compute clusters for faster training and inference. As AI models and datasets grow, the system can scale seamlessly to accommodate the increasing demands. Losing data due to storage failures can be catastrophic, so the storage is highly reliable to ensure data durability.

Fast storage access
The storage systems also need to provide fast read and write speeds to keep up with the demands of the training process. Otherwise, if storage access is slow, it can significantly hinder the training process.

Networking

Fast and efficient communication is essential for coordinating the work all of the processors in a high performance computing cluster are undertaking. Google's global fiber network provides high-bandwidth, low-latency connectivity, ensuring smooth data flow and efficient communication between different parts of the AI infrastructure.

Using Google Cloud infrastructure can benefit teams in several ways, particularly with respect to scalability, performance, and cost-effectiveness.

## AI on the edge

### Edge computing
While hosting infrastructure on the cloud is powerful, it's not always the ideal solution. Imagine a self-driving car needing to make split-second decisions. It can't wait for data to travel to the cloud and back. That's where edge computing comes in.

Google provides the tools to deploy your AI models in these different locations, giving you more control and flexibility.

Edge computing runs AI on devices or servers closer to the data source or point of need.

Why go local or edge?
Imagine a drone navigating a complex environment. It needs to react instantly to obstacles, making cloud processing too slow. Running AI locally on the drone ensures real-time responsiveness. Other benefits include increased data privacy and reduced reliance on internet connectivity.

To run powerful AI models on edge devices and mobile phones, Google provides tools like Lite Runtime (LiteRT). Think of LiteRT as a platform that helps machine learning models work efficiently on your device. To learn more about Google's high-performance runtime for on-device AI

#### Gemini Nano
Gemini Nano is Google's most efficient and compact AI model, specifically designed to run on the edge on devices like smartphones and embedded systems. It's part of the larger Gemini family of models. Think of Gemini Nano as a miniature version of the powerful AI that usually lives in Google's data centers. This "on-device" or edge approach offers several benefits.

Privacy
Your data stays on your device, enhancing your privacy.
Speed
You get fast responses since there's no need to send data to the cloud.
Offline access
Gemini Nano can work even without an internet connection.

Even when your goal is to deploy on the edge, Agent Platform provides a powerful platform for building, training, and refining your models. You can leverage Vertex AI's capabilities for tasks like data preparation, model training, tuning, model evaluation, collaboration, and MLOps.



Once your model is ready for the edge, Agent Platform offers tools to streamline the deployment process.


You can:
Convert models: Convert your models to Lite Runtime (LiteRT) for optimal performance on edge devices.
Package and deploy: Package your models and dependencies into containers for deployment on various edge hardware.
Manage and monitor: Manage your edge deployments, track their performance, and gather insights to improve your models over time.

## Gen AI project resources: People, cost, and time

Roles and responsibilities
It is important to understand the resources you have available to build out your solution. Let’s dig deeper into some of the roles and responsibilities at each layer of the stack. 

The AI stack is designed to support different roles with specific needs. Before jumping into building out a solution, ensure you have the proper expertise and talent for what you want to build. Business users can improve their tasks, developers can build custom AI solutions, and AI practitioners can develop and deploy advanced AI models responsibly and securely. This collaborative ecosystem allows your organization to leverage the full potential of AI across various business functions.

Cost
The pricing model of gen AI can vary based on which part of the gen AI landscape you are looking at or even which specific product or company you are using. It is important to have a realistic budget and understanding of the cost for your project.

When building gen AI solutions, you pay for three primary activities:

Training the model.
Deploying the model to an endpoint.
Using the model to make predictions.

Training and deploying models often involve paying for the compute time used as well as the storage for the training data and model outputs.

Pricing for using models
Use the arrow to learn about different use cases.

Usage-based
You pay for the amount you use, often measured in tokens or characters processed. This is common for APIs like Google’s PaLM & Gemini APIs.

Pricing metrics for using models
Here are some metrics you may see for usage-based pricing models.

Use the arrow to learn about different use cases.

Tokens
A token represents a piece of text, like a word or part of a word.

Factors affecting cost 
Here are some factors affecting cost when using models.

Use the arrow to learn about different use cases.

Model size and complexity
Larger, more capable models generally cost more.

Time

It probably won’t surprise you to learn that the more custom your solution is, the more time and resources it takes to build. Using a prebuilt gen AI application takes seconds. Building your own custom solution with a custom AI model, can take months. Choosing to build a custom agent with AI Applications takes much less time. 

Think about your project timelines and evaluate them against your needs and requirements. Determine if the timeline is realistic based on your requirements.

Key takeaway
Successful generative AI initiatives require leaders to strategically allocate resources, including the right mix of business users, developers, and AI practitioners.  

Understanding the costs associated with different generative AI solutions, from pre-built applications to custom models, is crucial for effective budgeting. Aligning project timelines with the complexity of the chosen solution ensures realistic expectations and successful outcomes.

Gen AI solution needs
Before diving headfirst into your generative AI project, it's crucial to take a step back and assess your needs. Aligning your ambitions with your capabilities is key to setting realistic expectations and ensuring a successful outcome. Let's explore the key factors to consider when evaluating the scope and feasibility of your generative AI initiatives.

Scale
Is this for individual use, a small team, a large company, or millions of customers?

Small scale
As an individual user or small team, you can go very far with pre-built tools. Try to leverage existing gen AI powered applications.

Large scale
When building out solutions for millions of customers, you'll likely need a more robust and customized solution. Pick solutions that offer scalability and security. Factor in details to your decision like infrastructure costs, data storage, and potential latency challenges.

Customization
How specialized does your solution need to be? Powerful foundation models are constantly improving, making custom model development less necessary in many cases.


Start with existing models
Explore pre-trained models available through APIs or open-source libraries. Fine-tune these models on your specific data to achieve better performance for your tasks.

Identify your unique needs
What sets your project apart? Does it require specialized knowledge, handle complex tasks, or demand a unique user experience? This will help you determine the level of customization needed.

Consider data specificity
If your project deals with specialized domains like law or medicine, consider fine-tuning with domain-specific datasets or exploring models specifically trained for those areas.

Consider task complexity
Think about the complexity of the task itself. Are you aiming for simple tasks like text summarization or more intricate ones like code generation or creative writing? This influences model choice and training approaches.

User interaction
Think about how users will interact with your AI and design an intuitive and engaging experience.

User interface (UI)
Seamlessly integrate the AI into your existing workflows. This might involve a dedicated interface (e.g., a chatbot widget on your website) or embedding AI capabilities within your current applications.

Privacy
How sensitive is the data you are working with?

Data security
What measures will you implement to protect data during processing and storage? Implement robust security measures to protect data during processing and storage. Consider encryption, access controls, and secure data centers.

Latency
What are the acceptable response time limits?
Tip: Consider real-time requirements. Do you need instantaneous responses, or can you tolerate some delay? If real-time interaction isn't critical, you have more flexibility in choosing models and infrastructure.

Connectivity
Will the solution always have internet access?
Tip: Consider offline functionality. Are there scenarios where the solution needs to function without internet access?

Accuracy
How crucial is the accuracy of the AI's output? What are the acceptable error tolerances?
Tip: Define acceptable tolerances. Determine the level of accuracy required for your AI's output. This will influence model selection, training data, and evaluation metrics.

Do you need to understand the reasoning behind the AI's decisions? 
Tip: Transparency is key. In certain domains, understanding the AI's reasoning is crucial. Consider using models or techniques that offer explainability features, especially in healthcare or finance.


Comparing different companies and models
Evaluate model capabilities: Look for benchmarks and comparisons to assess the quality and performance of different models.
Compare pricing structures: Consider your expected usage and calculate the costs based on the pricing models and metrics of each provider.

Factor in additional costs: Account for potential costs like data storage, API calls, and fine-tuning.

Read the fine print: Pay attention to usage limits, data privacy policies, and other terms of service.

Model monitoring and retraining
Establish a system to continuously monitor your model's performance and retrain it periodically with updated data.

Data updates
Plan for regular data updates to keep your model fresh and relevant. This might involve adding new data sources, cleaning existing data, or adapting to evolving data formats.

Software updates and bug fixes
Stay informed about updates to your chosen AI platform, libraries, or frameworks, and implement them promptly to ensure optimal performance and security.

Hardware and infrastructure
Consider the maintenance needs of your hardware and infrastructure. This includes server maintenance, security updates, and capacity planning to accommodate growing data and user demands.

Security and compliance
Maintain a vigilant approach to security. Regularly review and update your security measures to protect against evolving threats. Ensure ongoing compliance with data privacy regulations as they change.

# Gen AI: Navigate the Landscape download

## Prompting techniques

Zero-shot prompting is like asking a foundation model to complete a task with no prior examples, relying solely on its existing knowledge. 
One-shot prompting involves showing the foundation model just one example, allowing it to learn and apply that knowledge to similar situations. 
Few-shot prompting, on the other hand, provides the foundation model with multiple examples to learn from, which helps it better understand the task and improve its performance.

### Role prompting
Role prompting is a technique used to guide the behavior of LLMs by assigning them a specific role or persona. This can be anything from a business analyst or Shakespearean actor to a helpful customer service agent. By instructing the model to adopt a particular role, you influence the style, tone, and focus of its responses.

By assigning a specific role or persona to a model, role prompting allows you to precisely influence the tone, style, and focus of its responses. This powerful technique effectively guides the AI to act as an expert or a character, helping you generate tailored content that perfectly matches your needs.

### Prompt chaining
When working in a tool like Gemini, it's important to understand how previous prompts may impact future prompts.

Prompt chaining is a powerful technique for getting more complex and nuanced results from large language models like Gemini. It's like having a conversation with the AI where each response builds upon the previous one, leading to a more sophisticated and refined outcome. 

This is where the concept of a chatbot comes in. You can have a back and forth with Gemini.

### Gems
Think of Gems as your personalized AI assistants within Gemini.

Personalized responses
Gems can be tailored with specific instructions and information. This allows them to provide responses that are more relevant to your needs and preferences within that particular use case.

Streamlined workflows
Gems can help you streamline repetitive tasks by providing templates, prompts, or guided interactions. This can save you time and effort when working on familiar projects or activities.

Reset context
You can set the context for a Gem, like giving it background information or specific instructions. Then, you can start multiple chats with that Gem, each with its own focus and flow. These chats remain separate, so information from one won't spill over into another. It's like having different conversations with the same expert, each tailored to a specific purpose.

You can choose to use Google’s existing Gems that were built for many different common tasks, or you can create your own gem tailored to your use case.

What information can you provide to a custom Gem?
When you create a custom Gem, you provide it with instructions on how to act, what to do, and more. You are essentially prompting the agent or Gem on how to act. It's time to take all that prompting practice to the next level: Prompting to build your agents.

This is a very good place to use role-prompting, where you prompt the Gem to act as though it is in a specific type of role, such as an air traffic controller. 

You can also provide specific documents and resources to a Gem, giving it a focused knowledge base to draw from. Imagine a "code reviewer" Gem that has access to your project's coding style guidelines and best practices documentation. This would allow the Gem to provide feedback grounded in your specific project requirements.

### Grounding
Grounding refers to the ability of the AI model to connect its output to verifiable and specific sources of information. 

Rag: Retrieval-augmented generation
One powerful grounding technique is retrieval-augmented generation (RAG). It involves:

Retrieving relevant information: The AI model first retrieves relevant information from a vast knowledge base (like a database, a set of documents, or even the entire web). This retrieval process is often powered by sophisticated techniques, like semantic search or vector databases.
Generating output: The model then uses this retrieved information to generate the final output. This could be anything from answering a question to writing a creative story.

RAG empowers AI models to go beyond simply memorizing information and instead learn to effectively access and utilize external knowledge sources to generate more insightful and reliable outputs.

Improved accuracy and relevance: By accessing and utilizing relevant information directly, RAG models can produce more accurate and informative outputs.
Improved explainability and transparency: RAG can increase transparency and trust in the AI system by showing the specific sources used to generate the output, ensuring that claims can be checked for accuracy. 

Extending LLM capabilities: RAG can constrain the output from an LLM so that it only generates a response based on the specific context it was provided. 

One way to start using a RAG system without any coding or database development is with a tool called Gemini Notebook.

### Gemini Notebook
Gemini Notebook is an AI-first notebook, grounded in your own documents, designed to help you gain insights faster. 

It’s a unique tool that is built using Gemini models and is designed to help you learn and understand information better by acting as a virtual research assistant. You can "ground" Gemini Notebook in specific sources like documents, presentations, or even audio and video files. This means your AI assistant will only use information from those sources to answer your questions and generate summaries.

Gemini Notebook Plus

For users who want to take their research to the next level, there's Gemini Notebook Plus, a subscription plan that offers benefits like increased capacity, customization of response length and style, and usage analytics. 

Gemini Notebook Enterprise

Gemini Notebook Enterprise takes Gemini Notebook Plus to the next level by adding compliance and administrative features necessary for enterprise environments. This includes extra privacy and security features to give you enhanced control over your data. Your notebook information is only shared with your chosen collaborators, and you can manage access using predefined identity and access management (IAM) roles. To learn more, see the documentation for What is Gemini Notebook Enterprise?.

Here's how it works
Gemini Notebook operates on the concept of "notebooks" to help you organize and interact with your information. Here's a breakdown:

Creating a notebook
Think of a notebook as a container for a specific project, topic, or area of interest.

You start by giving your notebook a title and then uploading the relevant source materials (documents, PDFs, even audio or video files) that you want it to learn from.

The chat interface
Within each notebook, you have a single, ongoing chat with the Gemini Notebook AI.

You can ask questions, request summaries, or give instructions, just like you would in a regular chat.

Gemini Notebook will respond using only the information from the sources you've uploaded to that specific notebook.

Saving insights as notes
When the AI provides a response you find valuable, you can save it as a note within the notebook.

These notes become an organized collection of key takeaways, summaries, or important points.

Sharing and collaboration
You can create "shared notebooks" to collaborate with others.

In a shared notebook, everyone can access the same sources, ask questions, and contribute to the notes.

This fosters collaborative learning and knowledge sharing.

How is Gemini Notebook different from Gemini and Gems?
While Gems excel at general tasks and workflows, Gemini Notebook is designed to help you deeply understand and learn from specific sets of information. It's like having a dedicated expert who has thoroughly analyzed your chosen materials and is ready to answer any question you have.

Gemini Notebook versus Gems
Hyper-focused knowledge: Instead of drawing from a broad knowledge base, Gemini Notebook focuses solely on the sources you provide. This could be anything from research papers and articles to meeting notes and presentations.
Interactive learning: Gemini Notebook goes beyond simply summarizing information. It encourages active learning by allowing you to ask questions, generate different types of summaries, and even create quizzes to test your understanding.
Source-based answers: Every answer and insight provided by Gemini Notebook is directly grounded in your uploaded sources. This ensures accuracy and allows you to easily trace back to the original information. If you ask Gemini Notebook a question that isn't covered in the materials you've provided, it will honestly tell you that it can't answer. It won't invent information or speculate. This ensures that the information you get is always grounded in your sources and reliable.

Use Cases:
Create training materials and documentation
Onboard new team members and facilitate knowledge transfer by creating a central repository for essential resources. Create a dedicated notebook to share with new hires with relevant onboarding materials (employee handbook, product documentation, training presentations). The new employees can then use Gemini Notebook to ask questions, clarify information, and gain a deeper understanding of their role and the company.

Researching a new topic
Gather articles and videos on a subject you're interested in and have Gemini Notebook create a comprehensive summary or answer specific questions you have.

Learn by listening to audio
With audio overviews in Gemini Notebook you can learn about the material covered in your sources through a podcast-like experience. Gemini Notebook will  summarize your material, connecting topics, and produce a downloadable audio with back-and-forth conversation between two AI hosts.

Preparing for a presentation
Feed Gemini Notebook your presentation slides and ask it to generate speaker notes or practice questions to help you rehearse.

Summarize documentation
Investment firms and legal teams need to thoroughly analyze documents and data. Upload financial statements, legal documents, and market research reports into Gemini Notebook. The team can then use the AI to quickly identify key information, summarize findings, and flag potential risks or inconsistencies.

Project proposals and plans
Keep everyone aligned and informed by creating a shared Gemini Notebook with all project-related information.

Key takeaway
Gemini Notebook is an AI-powered notebook designed to help you quickly gain insights from your documents. It acts as a virtual research assistant, able to summarize information, answer your questions, and even create different types of summaries based on your uploaded documents, presentations, or audio and video files. 

Unlike general AI tools that draw from a broad knowledge base, Gemini Notebook focuses solely on the sources you provide. This "grounding" ensures that the information it provides is accurate, relevant, and traceable back to your original materials.






