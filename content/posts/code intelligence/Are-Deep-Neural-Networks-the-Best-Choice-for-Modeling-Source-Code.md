---
title: "Are Deep Neural Networks the Best Choice for Modeling Source Code"
author: "Haobo Gu"
tags: [paper-note, code-completion]
date: 2020-02-19T17:28:31+08:00
summary: 
draft: true
---

## Background

Modeling source code is similar with natural language modeling. However, there are several key differences between modeling source code and natural language:

- Unlimited Vocabulary

  => need a model can process unlimited vocab

- Nested, Scoped, Locality

  Vocabulary scopes are nested

  => needs nested model, which can be re-estimated fast by the lower scope

- Dynamism

  Language model for source code must rapidly adapt to the working context

  => needs rapid estimation and re-estimation

  

