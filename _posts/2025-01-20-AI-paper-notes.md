The following are notes while reading papers in the [2025 AI Engineer Reading List](https://www.latent.space/p/2025-papers) They are personal notes and are not refined for an external audience really. Just in case it's helpful for someone else. 

{:toc}

## **Frontier Models**

- **Improving Language Understanding by Generative Pre-Training** (GPT-1) (2018) by OpenAI
    - Introduces GPT-1 (117M params).
    - Key idea: Pre-train + fine-tuning. Use **unsupervised** learning to first pre-train a generative model on a large corpus of text, and then fine-tune this model on specific downstream tasks with **supervised** learning. This paper presents a foundational framework for transfer learning in NLP, where pre-trained models can be fine-tuned on specific tasks to achieve significant improvements in performance.
- **Language Models are Unsupervised Multitask Learners** (GPT-2) (2019) by OpenAI
    - Introduces GPT-2 (1.5B params)
    - Key idea: GPT-2 performs well on a variety of language tasks without needing task-specific training data. One of the innovative aspects of GPT-2 is its ability to adapt to diverse tasks simply by conditioning on task-specific **prompts**.
        - When prompted with a question, it can generate a coherent answer based solely on its pre-training knowledge. For example, when evaluated on reading comprehension question-answering task, GPT-2 produces answer when conditioned on the reading material and a final token `A:` (Section 3.5), with greedy decoding. When evaluated on text summarization task, GPT-2 produces answer when conditioned on the article and the text `TL;DR:` .
        - When given a few examples within the prompt, the model can adapt its outputs to align closely with the demonstrated behavior in those examples. For example, when evaluated on machine translation task, GPT-2 produces translation when conditioned on examples pairs of the format `english sentence = french sentence` and then after a final prompt of `english sentence =`, with greedy decoding.
- **Language Models are Few-Shot Learners** (GPT-3) (2020) by OpenAI
    - Introduces GPT-3 (175B params)
    - Key idea: the GPT-3 paper demonstrated that **scaling up** LLMs greatly improves few-shot performance, **without extensive fine-tuning**. GPT-3 is trained without fine-tuning and performs tasks purely based on prompting.
        - Model performance improves with prompting, and with the number of examples in the model’s context. Few-shot learning also improves dramatically with model size.
        - In-context learning: providing inference-time demonstrations to the model. The terms “zero-shot”, “one-shot”, and “few-shot” refer to the number of demonstrations provided at inference time.
        - The models scale up well and didn’t hit the ceiling yet.
- **Evaluating Large Language Models Trained on Code** (Codex) (2021) by OpenAI
    - Key idea: Codex models are GPT models fine-tuned on code to complete coding tasks. The paper focuses on generating standalone Python functions from docstrings. The correctness of the generated code is evaluated automatically against unit tests.
        - Models
            - Codex is a GPT model fine-tuned on publicly available code from GitHub.
            - Codex-S, standing for “supervised fine-tuned”, is further trained on a dataset of standalone, correctly-implemented Python functions.
            - Codex-D is trained on pairs of function signatures, reference solutions, and corresponding docstrings. The model focuses on generating docstrings from given implementation.
        - Codex uses repeated sampling from the model. By generating multiple code samples for a given prompt, the chance of obtaining at least one solution increased significantly.
        - Pass@k evaluation metrics: the probability of a model generating at least one correct solution within k samples.
- **Training language models to follow instructions with human feedback** (InstructGPT) (2022) by OpenAI
    - Key: After supervised fine-tuning, use RLHF to align the model.
        - Misalignment: the objective function of LLM - predicting the next word - is different from the objective of “following user instructions carefully and safely and helpfully”. Thus, the LLM objective is *misaligned*.
        - RLHF:
            - Step 1: Collect demonstration data of desired behavior, and perform supervised fine-tuning.
            - Step 2: Collect comparison data of model outputs and human ranking, to train a reward model.
            - Step 3: Fine-tune the model with RL, using PPO algorithm.
        - Alignment tax occurs when an alignment procedure negatively impacts performance on certain tasks. The author discussed mitigating this by adding pretraining updates to the PPO fine-tuning process.
    - Aside: The initial ChatGPT model is a sibling model of InstructGPT.
    
    ![image.png](../assets/images/sft_rlhf.png)
    
- **GPT-4 Technical Report** (GPT-4) (2023) by OpenAI
    - Key Idea: GPT-4 is a **multimodal** model capable of processing both image and text input to generate text outputs. The paper describes methods for **reliably predicating** model capabilities and mitigating safety risks through reinforcement learning from human feedback (**RLHF**).
        - Predictable scaling: Training LLM is expensive. OpenAI created infrastructure and optimization methods with consistent behavior across various scale. Model capabilities is predicated by fitting a power law.
        - RLHF: Simply increasing the size of language models doesn’t guarantee improved alignment with user intent. LLMs can still produce false, toxic, or unhelpful outputs. RLHF is used to fine-tune the model. This involves collecting demonstration data of desired behavior and comparison data where human labelers rank different outputs.
            - RLHF significantly enhances the models’s safety by reducing the likelihood of generating harmful content and improving the model’s ability to follow user instructions. However, RLHF doesn’t appear to significantly improve model’s capabilities. Evaluation shows that the base GPT-4 and post-RLHF model achieve similar scores on some benchmarks.
- [GPT-4o](https://openai.com/index/hello-gpt-4o/) (2024) by OpenAI
    - Prior to GPT-4o, ChatGPT supported voice through a pipeline of three separate models: audio to text → GPT-4 → text to audio. This had high latency, and GPT-4 lost a lot of information because it couldn’t directly observe tone, multiple speakers or background noise, and it couldn’t output laughter, singing or express emotion.
    - GPT-4o is a trained end-to-end across text, vision, and audio, meaning that all inputs and outputs are processed by the same model.
- [GPT-o1](https://openai.com/index/introducing-openai-o1-preview/), [technical report](https://openai.com/index/learning-to-reason-with-llms/), [system card](https://openai.com/index/openai-o1-system-card/) (2024) By OpenAI
    - GPT-o1 spends more time thinking before responding. It’s ”a significant advancement for complex reasoning tasks and represents a new level of AI capability”. O1 ranks 89th percentile on Codeforces, among top 500 students in USA Math Olympiad, and exceeds human PhD-level on a benchmark of physics, biology, and chemistry problems.
    - O1 uses chain of thought to think longer before responding. The chain of thought is hidden from the user though.
    - Compared with GPT-4o, o1 is slightly worse at text writing, but better at programming, data analysis, and math.
- **Deliberative Alignment: Reasoning Enables Safer Language Models** (GPT-o3) (2024) by OpenAI
    - TL;DR: Instead of aligning the model using human-labelled outputs and RLHF that implicitly encodes safety specifications, ask the model to reason through safety specifications explicitly with CoT and then perform RL. The complex reasoning ability, enabled by CoT, is foundational here.
    - Previously, LLMs are aligned using Supervised Fine Tuning (SFT) and Reinforcement Learning from Human Feedback (RLHF). There are two limitations with this approach. First, LLMs respond instantly without deliberation even in complex safety scenarios. Second, LLMs infer safety standards indirectly from labelled examples, instead of directly learning the safety specifications.
    - Deliberative Alignment addresses the two issues by teaching LLM to explicitly reason through safety specifications before producing an answer, using the chain-of-thought (CoT) reasoning ability introduced in GPT-o1.
    - The method involves two core stages, integrating process-based and outcome-based supervision. The training procedure requires **no human-labeled completions**.
        - In the first stage, teach the model to reason through safety specifications with CoT using supervised fine-tuning on `(prompt, CoT, output)`. The dataset is constructed by prompting another o-type model with the safety specifications, generating model completions, and stripping away the safe specifications from the prompt.
        - In the second stage, use RL to train the model. The reward model is a judge LLM that is given the safety specifications.
- **The Claude 3 Model Family: Opus, Sonnet, Haiku** (Claude 3) (2024) by Anthropic
    - Not particularly interesting
- “Gemini: A Family of Highly Capable Multimodal Models” (Gemini 1) (2023) by DeepMind
    - Not particularly interesting
- **LLaMA: Open and Efficient Foundation Language Models** (LLaMA 1) (2023) by Meta
    - The objective is to train a series of language models that deliver optimal performance across different **inference** budgets. Smaller models trained on more data can be more efficient at inference, making them cheaper to run in real-world applications, despite potentially longer training times.
    - LLaMA models are trained on more tokens than typically used and achieve performance similar to other larger models like GPT-3, Chinchilla and PaLM. LLaMa 1 is trained exclusively on public data.
- **LLaMA 2: Open Foundation and Fine-Tuned Chat Models** (LLaMA 2) (2023) by Meta
    - Fine-tunes for chat: LLaMA 2-Chat
    - Llama 2 uses a technique called "Ghost Attention" (GAtt), which involves augmenting user messages with instructions throughout the dialogue history during fine-tuning. This allows the model to consistently adhere to instructions over multiple turns. E.g. “Ack like Washington”.
- **The Llama 3 Herd of Models** (LLaMA 3) (2024) by Meta
    - LLaMa 3 405B. The paper identified three key levers to develop LLM:
        - Data: Improved both quantity and quality. LLaMA 3 was trained on 15T tokens, compared to 1.5T  for Llama 2.
        - Scale: Trained at a larger scale with 50x FLOPs than Llama2.
        - Managing complexity: To improve scaling, use supervised finetuning (SFT), rejection sampling (RS) and direct preference optimization (DPO), instead of RLHF with PPO.
    - Shared details about the development process. Dataset construction, large-scale training, model architecture, multimodality support, etc.
    - Maybe worth a more in-depth read?
- **DeepSeek LLM: Scaling Open-Source Language Models with Longtermism** (DeepSeek-v1) (2024)
    - Explores scaling law of hyperparameters. Proposes new scaling law variant. Argues that scaling behavior is affected by the quality of dataset.
- **DeepSeek-V2: A Strong, Economical, and Efficient Mixture-of-Experts Language Model** (DeepSeek-v2) (2024)
    - DeepSeek-v2 provides economical training and efficient inference. It replaces traditional multi-head attention with multi-head latent attention to reduce KV cache, and uses MoE to reduce training cost. With only 21B activated params, it achieves similar performance with Llama 70B.
    - MLA: See [MHA vs. MQA vs. GQA vs. MLA](https://www.notion.so/MHA-vs-MQA-vs-GQA-vs-MLA-1818cacb1fea804cb3cbe40ee55ff31a?pvs=21)
- **DeepSeek-v3 Technical Report** (DeepSeek-v3) (2024)
    - On top of DeekSeek-v2’s, adds an auxiliary-loss-free load balancing to DeekSeekMoE, and Multi-Token Prediction (MTP). Cost-effective due to using FP8 mixed precision training and meticulous engineering optimizations in the training framework. Achieved economical efficiency and strong performance.
        - MoE model performance suffer from unbalanced expert load. Auxiliary loss is a common solution but it could impair model performance. DeepSeek-v3 introduces a bias term and dynamically adjust it during training time to encourage load balancing.
        - MTP: Introduced by Meta. Predict multiple tokens instead of a single one at each position.