.. LangFair documentation master file, created by
   sphinx-quickstart on Wed Jun 12 09:11:05 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.
   
Welcome to LangFair's documentation!
====================================

LangFair is a comprehensive Python library designed for conducting use-case-specific bias and fairness assessments for large language model (LLM) use cases. Using a unique Bring Your Own Prompts (BYOP) approach, LangFair helps you:

✨ **Evaluate Real-World Scenarios**: Evaluate bias and fairness for actual LLM use cases

🎯 **Get Actionable Metrics**: Measure toxicity, stereotypes, and fairness with applicable metrics

🔍 **Make Informed Decisions**: Use our framework to choose the right evaluation metrics

🛠️ **Integrate with Workflows**: Easy-to-use Python interface for seamless implementation

:doc:`Get Started → <usage>` | :doc:`View Examples → <notebooks/examples/index>`

Why LangFair?
-------------

Static benchmark assessments, which are typically assumed to be sufficiently representative, often fall short in capturing the risks associated with all possible use cases of LLMs. These models are increasingly used in various applications, including recommendation systems, classification, text generation, and summarization. However, evaluating these models without considering use-case-specific prompts can lead to misleading assessments of their performance, especially regarding bias and fairness risks.

LangFair addresses this gap by adopting a Bring Your Own Prompts (BYOP) approach, allowing users to tailor bias and fairness evaluations to their specific use cases. This ensures that the metrics computed reflect the true performance of the LLMs in real-world scenarios, where prompt-specific risks are critical. Additionally, LangFair's focus is on output-based metrics that are practical for governance audits and real-world testing, without needing access to internal model states.


Featured Resources
------------------

Check out our featured resources to help you get started with LangFair.

- 📝 `LangFair tutorial <https://medium.com/cvs-health-tech-blog/how-to-assess-your-llm-use-case-for-bias-and-fairness-with-langfair-7be89c0c4fab>`_ on Medium
- 💻 `Software paper <https://arxiv.org/abs/2501.03112v1>`_ on how LangFair compares to other toolkits
- 📖 `Research paper <https://arxiv.org/abs/2407.10853>`_ on our evaluation approach


-----------

.. toctree::
   :maxdepth: 1
   :caption: Contents:
   
   Get Started <usage>
   Choosing Metrics<choosing_metrics>
   API <api>
   /notebooks/examples/index
   Contributor Guide <guide>