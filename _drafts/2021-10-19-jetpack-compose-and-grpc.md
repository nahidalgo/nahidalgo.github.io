---
layout: post
title:  "Jetpack Compose and gRPC with Kotlin and Spring"
date:   2021-10-19 01:19:16 -0300
categories: android kotlin compose grpc
---
## Compose
Jetpack Compose is Android's new declarative API for building UI faster and more intuitively. The new Compose API makes it possible to create reactive stateful or stateless Components. It takes advantage of the other Jetpack tools and integrates seamlessly with ViewModels, Kotlin Coroutines, Kotlin Flow, LiveData, and Room. In this example, we will use some of these tools to build our example app.

## gRPC

## What are we building
We are building a simple real-time messaging app taking advantage of all the tools gRPC and Jetpack libraries provide. We'll be using gRPC streaming functionalities with the help of Kotlin Flow to send and receive our messages to all listening devices simultaneously in real-time. The Android client will be an app capable of sending messages through gRPC to our server running Spring. The server will receive the messages, save them and stream them to all the listening clients.

## Sample Compose Code