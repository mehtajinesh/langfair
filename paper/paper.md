---
title: 'LangFair: A Python Package for Assessing Bias and Fairness in Large Language Model Use Cases'
tags:
  - Python
  - Large Language Model
  - Bias
  - Fairness
authors:
  - name: Dylan Bouchard
    orcid: 0009-0004-9233-2324
    affiliation: 1
  - name: Mohit Singh Chauhan
    orcid: 0000-0002-7817-0427
    affiliation: 1  
  - name: David Skarbrevik
    orcid: 0009-0005-0005-0408
    affiliation: 1
  - name: Viren Bajaj
    orcid: 0000-0002-9984-1293
    affiliation: 1
  - name: Zeya Ahmad
    orcid: 0009-0009-1478-2940
    affiliation: 1
affiliations:
 - name: CVS Health Corporation
   index: 1
date: 1 June 2024
bibliography: paper.bib

---
# Summary
Large Language Models (LLMs) have been observed to exhibit bias in numerous ways, potentially creating or worsening outcomes for specific groups identified by protected attributes such as sex, race, sexual orientation, or age. To help address this gap, we introduce `langfair`, an open-source Python package that aims to equip LLM practitioners with the tools to evaluate bias and fairness risks relevant to their specific use cases.^[The repository for `langfair` can be found at https://github.com/cvs-health/langfair.] The package offers functionality to easily generate evaluation datasets, comprised of LLM responses to use-case-specific prompts, and subsequently calculate applicable metrics for the practitioner's use case. To guide in metric selection, LangFair offers an actionable decision framework, discussed in detail in the project's companion paper, @bouchard2024actionableframeworkassessingbias. 


# Statement of Need
Traditional machine learning (ML) fairness toolkits like AIF360 [@aif360-oct-2018], Fairlearn [@Weerts_Fairlearn_Assessing_and_2023], Aequitas [@2018aequitas] and others [@vasudevan20lift; @DBLP:journals/corr/abs-1907-04135; @tensorflow-no-date] have laid crucial groundwork. These toolkits offer various metrics and algorithms that focus on assessing and mitigating bias and fairness through different stages of the ML lifecycle. While the fairness assessments offered by these toolkits include a wide variety of generic fairness metrics, which can also apply to certain LLM use cases, they are not tailored to the generative and context-dependent nature of LLMs.^[The toolkits mentioned here offer fairness metrics for classification. In a similar vein, the recommendation fairness metrics offered in FaiRLLM [@Zhang_2023] can be applied to ML recommendation systems as well as LLM recommendation use cases.]

LLMs are used in systems that solve tasks such as recommendation, classification, text generation, and summarization. In practice, these systems try to restrict the responses of the LLM to the task at hand, often by including task-specific instructions in system or user prompts. When the LLM is evaluated without taking the set of task-specific prompts into account, the evaluation metrics are not representative of the system's true performance. Representing the system's actual performance is especially important when evaluating its outputs for bias and fairness risks because they pose real harm to the user and, by way of repercussions, the system developer.

Most evaluation tools, including those that assess bias and fairness risk, evaluate LLMs at the model-level by calculating metrics based on the responses of the LLMs to static benchmark datasets of prompts [@rudinger-EtAl:2018:N18; @zhao-2018; @webster-etal-2018-mind; @levy2021collecting; @nadeem2020stereoset; @bartl2020unmasking; @nangia2020crows; @felkner2024winoqueercommunityintheloopbenchmarkantilgbtq; @barikeri2021redditbiasrealworldresourcebias; @kiritchenko2018examininggenderracebias; @qian2022perturbationaugmentationfairernlp; @Gehman2020RealToxicityPromptsEN; @bold_2021; @huang2023trustgptbenchmarktrustworthyresponsible; @nozza-etal-2021-honest; @parrish-etal-2022-bbq; @li-etal-2020-unqovering; @10.1145/3576840.3578295] that do not consider prompt-specific risks and are often independent of the task at hand. Holistic Evaluation of Language Models (HELM) [@liang2023holisticevaluationlanguagemodels], DecodingTrust [@wang2023decodingtrust], and several other toolkits [@srivastava2022beyond; @huang2024trustllm; @eval-harness; @Arshaan_Nazir_and_Thadaka_Kalyan_Chakravarthy_and_David_Amore_Cecchini_and_Thadaka_Kalyan_Chakravarthy_and_Rakshit_Khajuria_and_Prikshit_Sharma_and_Ali_Tarik_Mirik_and_Veysel_Kocaman_and_David_Talby_LangTest_A_comprehensive_2024; @huggingface-no-date] follow this paradigm. 

LangFair complements the aforementioned frameworks because it follows a bring your own prompts (BYOP) approach, which allows users to tailor the bias and fairness evaluation to their use case by computing metrics using LLM responses to user-provided prompts. This addresses the need for a task-based bias and fairness evaluation tool that accounts for prompt-specific risk for LLMs.^[Experiments in @wang2023decodingtrust demonstrate that prompt content has substantial influence on the likelihood of biased LLM responses.]

Furthermore, LangFair is designed for real-world LLM-based systems that require governance audits. LangFair focuses on calculating metrics from LLM responses only, which is more practical for real-world testing where access to internal states of model to retrieve embeddings or token probabilities is difficult. An added benefit is that output-based metrics, which are focused on the downstream task, have shown to be potentially more reliable than metrics derived from embeddings or token probabilities [@goldfarbtarrant2021intrinsicbiasmetricscorrelate; @delobelle-etal-2022-measuring].



# Generation of Evaluation Datasets
The `langfair.generator` module offers two classes, `ResponseGenerator` and `Counterfactual`-`Generator`, which aim to enable user-friendly construction of evaluation datasets for text generation use cases.


### `ResponseGenerator` class
To streamline generation of evaluation datasets, the `ResponseGenerator` class wraps an instance of a `langchain` LLM and leverages asynchronous generation with `asyncio`. To implement, users simply pass a list of prompts (strings) to the `ResponseGenerator.generate_responses` method, which returns a dictionary containing prompts, responses, and applicable metadata.

### `CounterfactualGenerator` class 
In the context of LLMs, counterfactual fairness can be assessed by constructing counterfactual input pairs [@gallegos2024biasfairnesslargelanguage; @bouchard2024actionableframeworkassessingbias], comprised of prompt pairs that mention different protected attribute groups but are otherwise identical, and measuring the differences in the corresponding generated output pairs. These assessments are applicable to use cases that do not satisfy fairness through unawareness (FTU), meaning prompts contain mentions of protected attribute groups. To address this, the `CounterfactualGenerator` class offers functionality to check for FTU, construct counterfactual input pairs, and generate corresponding pairs of responses asynchronously using a `langchain` LLM instance.^[In practice, a FTU check consists of parsing use case prompts for mentions of protected attribute groups.] Off the shelf, the FTU check and creation of counterfactual input pairs can be done for gender and race/ethnicity, but users may also provide a custom mapping of protected attribute words to enable this functionality for other attributes as well.

# Bias and Fairness Evaluations for Focused Use Cases
Following @bouchard2024actionableframeworkassessingbias, evaluation metrics are categorized according to the risks they assess (toxicity, stereotypes, counterfactual unfairness, and allocational harms), as well as the use case task (text generation, classification, and recommendation).^[Note that text generation encompasses all use cases for which output is text, but does not belong to a predefined set of elements (as with classification and recommendation).] Table 1 maps the classes contained in the `langfair.metrics` module to these risks. These classes are discussed in detail below.


Class          | Risk Assessed             | Applicable Tasks       |    
---------------|---------------------------|------------------------|
`ToxicityMetrics` | Toxicity          |        Text generation
`StereotypeMetrics` | Stereotypes      |          Text generation 
`CounterfactualMetrics` | Counterfactual fairness | Text generation 
`RecommendationMetrics` | Counterfactual fairness | Recommendation  
`ClassificationMetrics` | Allocational harms     | Classification     

**Table 1** : Classes for Computing Evaluation Metrics in langfair.metrics



### Toxicity Metrics
The `ToxicityMetrics` class facilitates simple computation of toxicity metrics from a user-provided list of LLM responses. These metrics leverage a pre-trained toxicity classifier that maps a text input to a toxicity score ranging from 0 to 1 [@Gehman2020RealToxicityPromptsEN; @liang2023holisticevaluationlanguagemodels]. For off-the-shelf toxicity classifiers, the `ToxicityMetrics` class provides four options: two classifiers from the `detoxify` package, `roberta-hate-speech-dynabench-r4-target` from the `evaluate` package, and `toxigen` available on HuggingFace.^[https://github.com/unitaryai/detoxify; https://github.com/huggingface/evaluate; https://github.com/microsoft/TOXIGEN] For additional flexibility, users can specify an ensemble of the off-the-shelf classifiers offered or provide a custom toxicity classifier object. 

### Stereotype Metrics
To measure stereotypes in LLM responses, the `StereotypeMetrics` class offers two categories of metrics: metrics based on word cooccurrences and metrics that leverage a pre-trained stereotype classifier. Metrics based on word cooccurrences aim to assess relative cooccurrence of stereotypical words with certain protected attribute words. On the other hand, stereotype-classifier-based metrics leverage the `wu981526092/Sentence-Level-Stereotype-Detector` classifier available on HuggingFace [@zekun2023auditinglargelanguagemodels] and compute analogs of the aforementioned toxicity-classifier-based metrics [@bouchard2024actionableframeworkassessingbias].^[https://huggingface.co/wu981526092/Sentence-Level-Stereotype-Detector]

### Counterfactual Fairness Metrics for Text Generation
 The `CounterfactualMetrics` class offers two groups of metrics to assess counterfactual fairness in text generation use cases. The first group of metrics leverage a pre-trained sentiment classifier to measure sentiment disparities in counterfactually generated outputs (see @huang2020reducingsentimentbiaslanguage for further details). This class uses the `vaderSentiment` classifier by default but also gives users the option to provide a custom sentiment classifier object.^[https://github.com/cjhutto/vaderSentiment] The second group of metrics addresses a stricter desiderata and measures overall similarity in counterfactually generated outputs using well-established text similarity metrics [@bouchard2024actionableframeworkassessingbias].


### Counterfactual Fairness Metrics for Recommendation
The `RecommendationMetrics` class is designed to assess counterfactual fairness for recommendation use cases. Specifically, these metrics measure similarity in generated lists of recommendations from counterfactual input pairs. Metrics may be computed pairwise [@bouchard2024actionableframeworkassessingbias], or attribute-wise [@Zhang_2023].

### Fairness Metrics for Classification
When LLMs are used to solve classification problems, traditional machine learning fairness metrics may be applied, provided that inputs can be mapped to a protected attribute. To this end, the `ClassificationMetrics` class offers a suite of metrics to address unfair classification by measuring disparities in predicted prevalence, false negatives, or false positives. When computing metrics using the `ClassificationMetrics` class, the user may specify whether to compute these metrics as pairwise differences [@aif360-oct-2018] or pairwise ratios [@2018aequitas].

# Semi-Automated Evaluation

### `AutoEval` class
To streamline assessments for text generation use cases, the `AutoEval` class conducts a multi-step process (each step is described in detail above) for a comprehensive fairness assessment. Specifically, these steps include metric selection (based on whether FTU is satsified), evaluation dataset generation from user-provided prompts with a user-provided LLM, and computation of applicable fairness metrics. To implement, the user is required to supply a list of prompts and an instance of `langchain` LLM. Below we provide a basic example demonstrating the execution of `AutoEval.evaluate` with a `gemini-pro` instance.^[Note that this example assumes the user has already set up their VertexAI credentials and sampled a list of prompts from their use case prompts.]


```python
from langchain_google_vertexai import ChatVertexAI
from langfair.auto import AutoEval

llm = ChatVertexAI(model_name='gemini-pro')
auto_object = AutoEval(prompts=prompts, langchain_llm=llm)
results = await auto_object.evaluate()
```

Under the hood, the `AutoEval.evaluate` method 1) checks for FTU, 2) generates responses and counterfactual responses (if FTU is not satisfied), and 3) calculates applicable metrics for the use case.\footnote{The `AutoEval` class is designed specifically for text generation use cases. Applicable metrics include toxicity metrics, stereotype metrics, and, if FTU is not satisfied, counterfactual fairness metrics.} This process flow is depicted in Figure 1.
![AutoEval_flowchart](AutoEval_flowchart_colored.png)

**Figure 1**:Flowchart of internal design of Autoeval.evaluate method

# Author Contributions
Dylan Bouchard was the principal developer and researcher of the LangFair project, responsible for conceptualization, methodology, and software development of the `langfair` library. Mohit Singh Chauhan was the architect behind the structural design of the `langfair` library and helped lead the software development efforts. David Skarbrevik was the primary author of LangFair's documentation, helped implement software engineering best practices, and contributed to software development. Viren Bajaj wrote unit tests, contributed to the software development, and helped implement software engineering best practices. Zeya Ahmad contributed to the software development. 

# Acknowledgements

We wish to thank Piero Ferrante, Blake Aber, Xue (Crystal) Gu, and Zirui Xu for their helpful suggestions.


# References
