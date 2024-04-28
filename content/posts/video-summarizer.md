+++
title = 'Lessons from building an automated short video generator '
date = 2024-04-20T12:10:07-05:00
+++

# Intro
Last week I went to the MongoDB Generative AI Hackathon in NYC! It was a really fun experience, and I learned a lot about leveraging state-of-the-art AI to build incredible things. At the same time, I also learned about some limitations of the current machine learning space, and got some experience with nailing down the appropriate "scope" of a project meant to be built in 6 hours.
 Let's start with the idea.
Imagine you have a long soccer video, and you want to get a highlight clip from the video. What do you do? Here's the basic idea.

> Summarize the video --> Generate summary audio --> Get the right clips from the video, corresponding to the summary --> Stitch it all together for the final video

## 1. Summarizing the video
 Now for generating the summary, we have a couple of options.
1. Use a model trained on videos to directly get the description of the video
2. Get the image frames for the video, use an image captioning model to describe all the images, and fuse together the description to "understand" the video 
3. Use the commentary in the video to understand the most important things in it, and use the corresponding video segments.
---

Option 1 is not feasible, especially for a 6-hour hackathon. Video AI is still very much a work-in-progress, and finding models that suit our purpose while being easy to deploy is hard. Potential benefits of this approach would have been the reduction of data pre-processing requirements and its applicability to a wide range of videos without the need for specific adaptations. It could work just as well for both soccer clips and animal planet clips. If I was Google, option 1 would be tempting.

---
Coming to option 2, an immediate benefit that stood out to us was that this approach would work for "silent" videos as well, since it depends only on the images in the video. We started experimenting with this, using the BLIP model to caption the image. 
We ran into a few problems.
- The model describes the image accurately, but does not take video context into account. For example, a starting shot of a soccer match is described as "A foot next to a ball". This is correct - but it does not take into account the larger context. While this is expected of an image model, it still presented a challenge for our use case.
- We used a very simple method to divide the video into images, creating two snapshots per second of the video. The "diff" between the description of two subsequent images was often nil, leading to useless computation.
- Coming to computation, the BLIP models are _large_. Using HuggingFace inference API was too rate-limiting even for our PoC use case, deploying locally was impossible due to hardware constraints, and deploying to SageMaker was difficult due to being a student (I am poor)
- The BLIP models also often spit out hallucinatory descriptions. This is the most harmful of all the problems - since it messes up all the subsequent steps. 

While I still think this approach holds some promise, it would take significantly more effort than was possible in a 6-hour window. Trying some of these BLIP-related things out is how I learned a bit more about nailing the right scope for a project. While this method could have led to a significantly better output, it would have been a waste of time, given the short window. Reducing the scope by cancelling option 2 allowed us to focus on a solution that worked. 

---
Option 3: Using the commentary in the video to understand the most important things in it, and using the corresponding video segments. This is the option we ended up going with. It has a key limitation - this will simply not work for a video without any dialogue or sound. We were okay with this, since we decided to focus on soccer clips, but options 1 and 2 may be the only feasible solutions if silent videos are taken into account. 
For our use case, however, option 3 is fantastic. Instead of even depending on the actual audio, we could use Youtube's auto-generated subtitles in SRT format. This gives us the content in the video, in an easy-to-parse, compact format. The use of timestamps will be explained later. We collected all the content from the subtitles, and used ChatGPT to generate a summary. We then used Whisper to generate audio from the text summary, to eventually layer over the generated short video.

## 2. Getting the appropriate clips

### Background on embeddings:
This post mentions embeddings - here's a short primer that chatGPT gave me after some prompting experiments : 
Think of embeddings as the brains of our language models—they're like these super-smart representations of words that live in this magical vector space. These embeddings aren't just random numbers; they're like little capsules of meaning and context for each word. So, when our NLP models see words, they don't just see letters strung together; they see these rich, nuanced representations that help them understand language way better. It's kind of like giving our models a secret decoder ring for language! And hey, these embeddings make our models really good at things like understanding language nuances, figuring out synonyms, and even translating languages. They're pretty much the unsung heroes behind the scenes of all our favorite language-powered technologies!

There's so much literature out there on embeddings. Google away!

---

The second problem we had to solve was - how do we get the appropriate clips which correspond to the summary? Since the image captioning approach with BLIP was not working, embedding image captions with image name metadata was not an option. After doing some research, and discussing the issue at hand with Andriy, the founder at Nomic, we decided to use Nomic embeddings. These embeddings embed images and text in the same latent space, allowing us to "link" the two together. We used a very simple method to divide the video into images, creating two snapshots per second of the video. then, we could directly store Nomic embeddings generated from the snapshots into MongoDB, along with the associated timestamp of those snapshots. Here's what an example looks like :
| startTimeStamp | endTimeStamp | embedding         |
|----------------|--------------|-------------------|
| 1s             | 1.5s         | [0.46, 0.15, ...] |
| ...            | ...          | ...               |

Once all the images are stored, we use the embedding of summary text from part 1 as a query for vector search. and retrieve 'n' documents with embeddings nearest to the query vector. We are able to use text to query images because Nomic embeddings are multimodal - they embed both images and text in the same latent space. For our scope, we just took the 50 nearest images. For a real-world solution, this number can be made dynamic, perhaps depending on the length of the video, or the target short clip length. 

### 3. Putting it all together
We now have the two pieces needed to generate a short clip :
- summary audio
- most relevant clips from the original video

We sort the relevant clips by their timestamps, and merge all of them to create a video using ffmpeg. Then, once again using ffmpeg, we layer the generated audio on top of this video to produce the final output. We end up getting a nice clip with goals and key moments from the original soccer video!

### Some relevant links

**Our Code**  : https://github.com/herzo175/mongodb-apr-2024-hackathon

**A different approach to solve this problem** : https://aws.amazon.com/blogs/media/video-summarization-with-aws-artificial-intelligence-ai-and-machine-learning-ml-services/

**Video ML** : https://github.com/HuaizhengZhang/Awsome-Deep-Learning-for-Video-Analysis

**Example video used by us** : https://www.youtube.com/watch?v=h4m68r8kWAc
