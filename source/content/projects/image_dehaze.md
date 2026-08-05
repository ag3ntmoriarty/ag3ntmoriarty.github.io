---
title: "Image - DeHaze"
date: 2023-09-24T06:49:05+05:30
description: "
AI-powered real-time dehazing algorithm to enhance visibility in smoke-filled environments for improved rescue operations."
image: "../../pics/Projects/image_dehaze.png"
draft: false
---

{{< video youtube="ftlFrpD1ex8"  autoplay="true">}}
Showcasing the hazy view (left) and the dehazed view (right)

## **Project Overview**  
The **AI-ML Based Intelligent De-Smoking/De-Hazing Algorithm** is an advanced real-time image and video processing solution designed to enhance visibility in fire-prone environments. Developed as part of **Smart India Hackathon (SIH) 2023**, this project focuses on improving rescue operations by providing clear visuals of areas affected by smoke and haze, particularly in indoor fire hazards.  

{{< video youtube="CqyUP8Gyb9Y" autoplay="false">}}

## **Problem Statement (SIH1417)**  
Fire hazards create **dense smoke** that significantly **reduces visibility**, making rescue operations more difficult. Existing dehazing algorithms suffer from **high latency** and may not perform well in real-time conditions. Our project aims to develop an AI-powered **low-latency, high-performance dehazing algorithm** to address this challenge effectively.  

## **Key Objectives**  
 - Real-time dehazing for images and videos  
 - Enhancing existing state-of-the-art models  
 - Reducing model latency while maintaining accuracy  
 - Proposing novel model architectures  

## **Our Solution**  
We implemented **two major improvements** over existing dehazing models:  

**GridDehazeNet Enhancement**  
   - Integrated **Pixel Attention Mechanism** to improve dehazing quality  
   - Improved **SSIM score** to **0.9841** (original: 0.9836)  
   - Reduced **latency to 9.91 ms** (original: 9.905 ms)  

**MixDehazeNet Optimization**  
   - Introduced **Mix Structure Block**  
   - Incorporated **Dark Channel Prior, Wiener Filter, and Sparse Attention**  
   - Combined with **Object Detection** for better scene understanding  

## **Technical Approach**  
- **Datasets Used:**  
  - **Training:** RESIDE, HAZE 4K  
  - **Testing:** SOTS (500 indoor images)  
- **Tech Stack:**  
  - **Backend:** Python, Node.js, Express  
  - **Frontend:** React.js  

## **Real-World Applications**  
- Enhancing visibility in **fire rescue operations**  
- **Real-time dehazing** in **vehicles and aircraft**  
- **Search and rescue missions** in low-visibility conditions  

## **Future Enhancements**  
🔹 Integration of **Image Segmentation**  
🔹 Further **latency reduction** for real-time applications  
🔹 Deployment as a **scalable cloud-based API**  
🔹 Combining **MixDehazeNet with YOLOv5** for enhanced **object detection** in hazy conditions  

This project represents a **step forward in AI-powered real-time image enhancement**, with direct applications in safety, transportation, and disaster management.  

📂 **GitHub Repository**: [SIH2023-PixelEncoders](https://github.com/agntgalahad/SIH2023-PixelEncoders)  

