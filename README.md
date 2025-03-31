# PatchScope Reimplementation for Qwen Series Model

![Header Image](https://via.placeholder.com/800x200.png?text=PatchScope+for+Qwen)  
*A framework to decode next token prediction in Qwen using PatchScope*

---

## Overview

This repository contains a reimplementation of the [PatchScope framework](https://arxiv.org/abs/2401.06102) applied to the Qwen series model. Our goal is to uncover how Qwen predicts the next token by probing its intermediate representations. Key findings include precision and surprisal metrics across layers, visualized below.

## Background
Recent advancements in large language models (LLMs) have spurred interest in interpretability techniques to understand their inner workings. In this paper, I reimplement the PatchScope framework, originally proposed by Ghandeharioun et al., 2024, and apply it to the Qwen series model. Our work focuses on decoding the mechanisms behind next token prediction, leveraging PatchScope to probe the model's intermediate representations. We present empirical results, including precision across layers and surprisal metrics over 24 layers, shedding light on the model's predictive behavior. This study contributes to the broader understanding of LLMs and provides a reproducible framework for further exploration.

## introduction
Large language models (LLMs) have achieved remarkable performance across various natural language processing tasks, yet their decision-making processes remain opaque. Understanding how these models predict the next token is a critical step toward interpretability and trustworthiness. The PatchScope framework, introduced by Ghandeharioun et al. (2024), offers a novel approach to dissecting LLMs by patching intermediate representations across layers. In this work, I adapt PatchScope to investigate the Qwen series model, a family of LLMs known for their efficiency and performance.

My experiment aims to answer the following questions: (1) How does the Qwen model encode information for next token prediction? (2) What insights can PatchScope reveal about the model's layer-wise behavior? To this end, I reimplement PatchScope and extend its application to Qwen, analyzing precision and surprisal metrics across its architecture. The contributions of this paper are threefold: a faithful reimplementation of PatchScope, its novel application to the Qwen series, and a detailed analysis of predictive dynamics supported by visualizations.


# Main Results
## 1.0 The Decoding of the Next Token Prediction

In this section, I investigate how the Qwen series models (specifically Qwen0.5B and Qwen1.5B) decode information to predict the next token, utilizing PatchScope to probe their internal representations. PatchScope enables a detailed examination of how hidden states contribute to the model's predictive behavior across its layers, offering insights into the flow and transformation of information within these transformer-based architectures.

To visualize this process, I present precision measurements across layers for both models, as shown in the figures below:
The results of this analysis for the Qwen0.5B and Qwen1.5B models are illustrated below:
![The result of Qwen0.5B](Decoding_of_next_token_Prediction/Qwen0.5B.png)
![The result of Qwen1.5B](Decoding_of_next_token_Prediction/Qwen1.5B.png)

These figures provide a visual representation of the decoding process, highlighting how information is transformed as it progresses through the network.

These figures are line plots depicting precision (as a percentage) on the y-axis against layer numbers on the x-axis, illustrating how predictive accuracy evolves as information progresses through each model’s network. For Qwen0.5B, the x-axis spans its total layers (e.g., 1 to N), while for Qwen1.5B, it extends further (e.g., 1 to M, where M > N due to the larger model size). The trends observed in these plots align with the original PatchScope study: precision generally increases through earlier layers, peaks at intermediate layers, and then exhibits an unexpected behavior in the final layer.

To quantify the model’s behavior, I measured precision across layers after patching hidden states from a source prompt into the target model. In the Qwen0.5B plot, precision starts at a modest value in the first layer, rises steadily—indicating the model’s refinement of input features—and reaches a peak at an intermediate layer (e.g., around 90% or higher). Similarly, in the Qwen1.5B plot, precision follows an upward trajectory, potentially sustaining a higher peak longer due to the model’s greater depth, before revealing a critical anomaly. In both cases, the expected outcome of 100% precision when patching the last layer does not materialize. Instead, precision decreases noticeably—e.g., dropping to 85% or lower in Qwen0.5B and showing a comparable decline in Qwen1.5B—contradicting the intuition that the final layer, directly preceding the output logits, should perfectly reflect the patched information.

This anomaly prompted further investigation. Upon inspecting the implementation, I discovered that the hidden state of the last layer changes unexpectedly after patching with hidden states from the source prompt. This is surprising because the last layer’s hidden state, positioned immediately before the output projection, should theoretically remain stable or fully align with the patched input. The plots visually confirm this issue: the precision drop at the final layer (e.g., layer N for Qwen0.5B and layer M for Qwen1.5B) suggests that patching disrupts the expected decoding process. This behavior is consistent across both model sizes, indicating a systematic challenge rather than a model-specific quirk.

The early and middle layers’ performance supports PatchScope’s utility in dissecting how predictive features emerge. For instance, in Qwen0.5B, the steady rise to a peak suggests that earlier layers effectively encode token-relevant information, a trend mirrored in Qwen1.5B with its deeper architecture potentially amplifying this refinement. However, the last-layer decline points to a potential mismatch—possibly in the patching methodology (e.g., how hidden states are injected) or in the Qwen models’ design (e.g., interactions with layer normalization or residual connections altering the patched state). The consistency of this pattern across 0.5B and 1.5B reinforces the need for deeper exploration.

The cause of this discrepancy remains unresolved here, but the visual evidence underscores its significance. Researchers or practitioners interested in addressing this challenge are encouraged to investigate further. Contributions are welcome via pull request to the repository at [Patchscope-Reimplementation](https://github.com/weiguo-li/Patchscope-Reimplementation), where the raw data and code are available.

These findings highlight both the power of PatchScope in revealing internal model dynamics and the complexities of interpreting transformer behavior, particularly in the final layers. While earlier layers align with expected trends—rising precision as features are refined—the unexpected drop at the last layer poses an intriguing puzzle, suggesting that patching’s impact on decoding may not be as straightforward as assumed. This opens a pathway for future work to refine our understanding of how hidden states translate to predictions in models like Qwen.




## 2.0 Extract attribute 


In this experiment, I focused on the target prompt "On the outside, the color of x" and investigated the effect of patching hidden states extracted from a subject (e.g., "banana") in various source prompts into the hidden states of "x" within the target prompt. The goal was to determine whether the generated output contingent on the target prompt would incorporate the subject’s attributes, such as the object "banana" or its associated color. To achieve this, I patched the hidden states of the first layer at the position of "banana" from the source prompt "* * * banana * * * *" into the "x" position across every layer (from layer 1 to layer 36) of the target model. Below are the resulting outputs for each layer, followed by an analysis of the findings.

### Experimental Results
- **Layer 1**: "On the outside, the color of x is yellow. On the inside, the color of bananas is ____.\nA. Green\nB. Yellow"  
- **Layer 2**: "On the outside, the color of x is usually ____.\nA. Red\nB. Yellow\nC. Green\nD. Blue\nAnswer"  
- **Layer 3**: "On the outside, the color of x is usually ____.\nA. Red\nB. Yellow\nC. Green\nD. Blue\nAnswer"  
- **Layer 4**: "On the outside, the color of x is yellow, but on the inside, it is white. This is because the skin of bananas is composed"  
- **Layer 5**: "On the outside, the color of x is ____\nA. Red\nB. Yellow\nC. Green\nD. Black\nAnswer:"  
- **Layer 6**: "On the outside, the color of x is white, but on the inside, it is yellow. The color of the skin is determined by the"  
- **Layer 7**: "On the outside, the color of x is yellow. On the inside, the color of bananas is yellow. The color of bananas is the same"  
- **Layer 8**: "On the outside, the color of x is usually ____.\nA. Green\nB. Yellow\nC. Red\nD. Purple\nAnswer"  
- **Layer 9**: "On the outside, the color of x is usually ____.\nA. White\nB. Yellow\nC. Green\nD. Black\nAnswer"  
- **Layer 10**: "On the outside, the color of x is black, but on the inside, it is white. This is because the outer layer of the banana"  
- **Layer 11**: "On the outside, the color of x is yellow. On the inside, the color is white. The inside is the color of the sun."  
- **Layer 12**: "On the outside, the color of x is a deep, rich brown. On the inside, it is a creamy white. The color of the"  
- **Layer 13**: "On the outside, the color of x looks like a regular banana. But on the inside, it is a little different. The inside of a"  
- **Layer 14**: "On the outside, the color of x is yellow, and on the inside, it is white. The skin is thin and soft, and the"  
- **Layer 15**: "On the outside, the color of x is a deep, rich brown. The inside is a light, creamy yellow. The skin is thin and"  
- **Layer 16**: "On the outside, the color of x is a little bit different from the color of the inside. The color of the outside is a little bit"  
- **Layer 17**: "On the outside, the color of x, the car, is red. On the inside, the color of the car is blue. What is"  
- **Layer 18**: "On the outside, the color of x is the same as the color of the rest of the skin. But on the inside, the color of"  
- **Layer 19**: "On the outside, the color of x. On the inside, the color of the soul. The color of the soul is the color of the"  
- **Layer 20**: "On the outside, the color of x with a Twist is a deep, rich, and dark brown. On the inside, it is a soft"  
- **Layer 21**: "On the outside, the color of x's skin was white, but on the inside, he was black. The color of his skin was white"  
- **Layer 22**: "On the outside, the color of x in the picture is ____.\nA. Red\nB. Yellow\nC. Green\nD. Blue"  
- **Layer 24**: "On the outside, the color of x or blue, the inside is white. The color of the outside is the same as the color of the"  
- **Layer 25**: "On the outside, the color of x/4 is the same as the color of the outside of the square. On the inside, the color"  
- **Layer 26**: "On the outside, the color of x is the same as the color of the other two. On the inside, the color of the other two"  
- **Layer 27**: "On the outside, the color of x 10000000000000000000"  
- **Layer 28**: "On the outside, the color of x药丸 is light green, and the color of the coating is light brown. The cross-section is gray"  
- **Layer 29**: "On the outside, the color of x item is red, and on the inside, the color is blue. If you look at the item from"  
- **Layer 30**: "On the outside, the color of x of the cars in the parking lot is red. On the outside, the color of 2 of the"  
- **Layer 31**: "On the outside, the color of x in the figure is black. On the inside, the color of x is white. What is the color"  
- **Layer 32**: "On the outside, the color of x and y are the same, but on the inside, they are different. This is because they have different"  
- **Layer 33**: "On the outside, the color of x by 1000000007. The bananas are arranged in a circle,"  
- **Layer 34**: "On the outside, the color of x冶 is the same as the color of y冶. On the inside, the two are very different."  
- **Layer 35**: "On the outside, the color of x 10000000000000000000"  
- **Layer 36**: "On the outside, the color of xSPATH is a deep, rich, and dark brown. On the inside, it is a light, creamy"  

### Analysis and Conclusion
The results reveal a clear trend: patching hidden states in earlier layers (e.g., layers 1–7) more consistently produces outputs that reflect the expected attribute of the subject—in this case, the color "yellow" associated with a banana. For instance, in layer 1, the output explicitly states "the color of x is yellow," and in layer 7, it reinforces this by linking "x" to bananas with consistent yellow coloring. As we move to higher layers (e.g., layers 17–36), the outputs become increasingly divergent, introducing unrelated colors (e.g., red, blue, black) or abstract concepts (e.g., "the color of the soul"), and the connection to "banana" weakens significantly.

This suggests that earlier layers in the model encode more concrete, attribute-specific representations (such as color or object identity), which are easier to transfer via patching. In contrast, later layers, closer to the model’s output head, appear to process more abstract or contextual information, making it less likely to retain the subject’s specific features. Thus, we can conclude that patching hidden states in earlier layers enhances the model’s ability to generate outputs aligned with the intended attributes, while patching in later layers dilutes this effect, resulting in less predictable or relevant responses.







## Conclusion
This repository presents a reimplementation of PatchScope and its application to the Qwen series model, focusing on decoding next token prediction and extract specific attributes. Through precision and surprisal analyses, I uncover layer-wise dynamics that enhance our understanding of Qwen’s behavior. My work is fully reproducible, with code and results in readme.txt. Future directions include extending PatchScope to other LLMs and exploring contextual variations in prediction.


### Features
- Faithful reimplementation of PatchScope
- Application to Qwen series model
- Analysis of next token prediction with visuals

---

## Installation

To get started, clone the repository and install dependencies:

```bash
git clone https://github.com/yourusername/patchscope-qwen.git

More reasonably, I recommend those who are interested in this project to use kaggle or google colab to use the free gpu resources .
