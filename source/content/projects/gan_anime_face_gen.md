---
title: "GAN Anime Face Generator"
date: 2023-01-16T06:49:05+05:30
description: "This project utilizes a Deep Convolutional Generative Adversarial Network (DCGAN) to generate 64x64 pixel anime-style face images from random noise inputs. "
image: "../../pics/Projects/gan.png"
draft: false
---

📂 **GitHub Repository**: [GAN-Anime-Face-Generator](https://github.com/agntgalahad/GAN-Anime-Face-Generator)  

The GAN Anime Face Generator is a Deep Convolutional Generative Adversarial Network (DCGAN) designed to create 64x64 pixel anime-style face images from random noise inputs. Trained on a dataset of anime faces, the model comprises two primary components:

- **Generator**: This network takes random noise as input and generates new anime face images.

- **Discriminator**: This network evaluates images to distinguish between real images from the dataset and fake images produced by the generator.

Through adversarial training, where the generator aims to produce convincing images and the discriminator strives to identify fakes, the system learns to generate increasingly realistic anime faces over time.

The project includes a Jupyter Notebook for training and generating images, with the model trained on a local runtime. The dataset used for training is available at [https://www.kaggle.com/datasets/soumikrakshit/anime-faces](https://www.kaggle.com/datasets/soumikrakshit/anime-faces).

This project demonstrates the application of GANs in generating novel anime-style faces, highlighting the potential of deep learning in creative and entertainment industries. 
