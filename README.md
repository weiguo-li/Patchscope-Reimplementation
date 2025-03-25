# PatchScope Reimplementation for Qwen Series Model

![Header Image](https://via.placeholder.com/800x200.png?text=PatchScope+for+Qwen)  
*A framework to decode next token prediction in Qwen using PatchScope*

---

## Overview

This repository contains a reimplementation of the [PatchScope framework](https://arxiv.org/abs/2401.06102) applied to the Qwen series model. Our goal is to uncover how Qwen predicts the next token by probing its intermediate representations. Key findings include precision and surprisal metrics across layers, visualized below.

### Features
- Faithful reimplementation of PatchScope
- Application to Qwen series model
- Analysis of next token prediction with visuals

---

## Installation

To get started, clone the repository and install dependencies:

```bash
git clone https://github.com/yourusername/patchscope-qwen.git
cd patchscope-qwen
pip install -r requirements.txt
