Quickstart Guide
================
(Optional) Create a virtual environment for using LangFair
----------------------------------------------------------
We recommend creating a new virtual environment using venv before installing LangFair. To do so, please follow instructions `here <https://docs.python.org/3/library/venv.html>`_.

Installing LangFair
-------------------
The latest version can be installed from PyPI:

.. code-block:: console

   pip install langfair

Usage Examples
--------------
Below are code samples illustrating how to use LangFair to assess bias and fairness risks in text generation and summarization use cases. The below examples assume the user has already defined a list of prompts from their use case , ``prompts``.

Generate LLM responses
^^^^^^^^^^^^^^^^^^^^^^
To generate responses, we can use LangFair's ``ResponseGenerator`` class. First, we must create a ``langchain`` LLM object. Below we use ``ChatVertexAI``, but any of `LangChain's LLM classes <https://js.langchain.com/docs/integrations/chat/>`_ may be used instead. Note that ``InMemoryRateLimiter`` is to used to avoid rate limit errors.

.. code-block:: python

   from langchain_google_vertexai import ChatVertexAI
   from langchain_core.rate_limiters import InMemoryRateLimiter
   rate_limiter = InMemoryRateLimiter(
       requests_per_second=4.5,
       check_every_n_seconds=0.5,
       max_bucket_size=280,
   )
   llm = ChatVertexAI(
       model_name="gemini-pro",
       temperature=0.3,
       rate_limiter=rate_limiter
   )

We can use ``ResponseGenerator.generate_responses`` to generate 25 responses for each prompt, as is convention for toxicity evaluation.

.. code-block:: python

   from langfair.generator import ResponseGenerator
   rg = ResponseGenerator(langchain_llm=llm)
   generations = await rg.generate_responses(prompts=prompts, count=25)
   responses = [str(r) for r in generations["data"]["response"]]
   duplicated_prompts = [str(r) for r in generations["data"]["prompt"]]  # so prompts correspond to responses


Compute toxicity metrics
^^^^^^^^^^^^^^^^^^^^^^^^
Toxicity metrics can be computed with ``ToxicityMetrics``. Note that use of ``torch.device`` is optional and should be used if GPU is available to speed up toxicity computation.

.. code-block:: python

   # import torch # uncomment if GPU is available
   # device = torch.device("cuda") # uncomment if GPU is available
   from langfair.metrics.toxicity import ToxicityMetrics
   tm = ToxicityMetrics(
       # device=device, # uncomment if GPU is available,
   )
   tox_result = tm.evaluate(
       prompts=duplicated_prompts,
       responses=responses,
       return_data=True
   )
   tox_result['metrics']
   # Output is below
   # {'Toxic Fraction': 0.0004,
   # 'Expected Maximum Toxicity': 0.013845130120171235,
   # 'Toxicity Probability': 0.01}
   
Compute stereotype metrics
^^^^^^^^^^^^^^^^^^^^^^^^^^
Stereotype metrics can be computed with ``StereotypeMetrics``.

.. code-block:: python

   from langfair.metrics.stereotype import StereotypeMetrics
   sm = StereotypeMetrics()
   stereo_result = sm.evaluate(responses=responses, categories=["gender"])
   stereo_result['metrics']
   # Output is below
   # {'Stereotype Association': 0.3172750176745329,
   # 'Cooccurrence Bias': 0.4476633365427837,
   # 'Stereotype Fraction - gender': 0.08}


Generate counterfactual responses and compute metrics
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
We can generate counterfactual responses with ``CounterfactualGenerator``.

.. code-block:: python

   from langfair.generator.counterfactual import CounterfactualGenerator
   cg = CounterfactualGenerator(langchain_llm=llm)
   cf_generations = await cg.generate_responses(
       prompts=prompts, attribute='gender', count=25
   )
   male_responses = [str(r) for r in cf_generations['data']['male_response']]
   female_responses = [str(r) for r in cf_generations['data']['female_response']]

Counterfactual metrics can be easily computed with ``CounterfactualMetrics``.

.. code-block:: python

   from langfair.metrics.counterfactual import CounterfactualMetrics
   cm = CounterfactualMetrics()
   cf_result = cm.evaluate(
       texts1=male_responses,
       texts2=female_responses,
       attribute='gender'
   )
   cf_result['metrics']
   # Output is below
   # {'Cosine Similarity': 0.8318708,
   # 'RougeL Similarity': 0.5195852482361165,
   # 'Bleu Similarity': 0.3278433712872481,
   # 'Sentiment Bias': 0.00099471451876019657}


Alternative approach: Semi-automated evaluation with ``AutoEval``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To streamline assessments for text generation and summarization use cases, the ``AutoEval`` class conducts a multi-step process that completes all of the aforementioned steps with two lines of code.

.. code-block:: python

   from langfair.auto import AutoEval
   auto_object = AutoEval(
       prompts=prompts,
       langchain_llm=llm,
       # toxicity_device=device # uncomment if GPU is available
   )
   results = await auto_object.evaluate()
   results['metrics']
   # Output is below
   # {'Toxicity': {'Toxic Fraction': 0.0004,
   #   'Expected Maximum Toxicity': 0.01384513012017123,
   #   'Toxicity Probability': 0.01},
   # 'Stereotype': {'Stereotype Association': 0.3172750176745329,
   #   'Cooccurrence Bias': 0.4476633365427837,
   #   'Stereotype Fraction - gender': 0.08,
   #   'Expected Maximum Stereotype - gender': 0.6035516738891,
   #   'Stereotype Probability - gender': 0.27036},
   # 'Counterfactual': {'male-female': {'Cosine Similarity': 0.8318708,
   #   'RougeL Similarity': 0.5195852482361165,
   #   'Bleu Similarity': 0.3278433712872481,
   #   'Sentiment Bias': 0.00099471451876019577}}}
