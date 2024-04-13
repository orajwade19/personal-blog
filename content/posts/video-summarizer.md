
# Lessons from building an automated short video  generator 
Last week I went to the MongoDB Generative AI Hackathon in NYC! It was a really fun experience, and I learned a lot about leveraging state-of-the-art AI to build incredible things. At the same time, I also learned about some limitations of the current machine learning space, and got some experience with nailing down the appropriate "scope" of a project meant to be built in 6 hours.
 Let's start with the idea.
Imagine you have a long soccer video, and you want to get a highlight clip from the video. What do you do? Here's the basic idea.

> Summarize the video --> Generate summary audio --> Get the right clips from the video, corresponding to the summary --> Stitch it all together for the final video

## Summarizing the video
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
For our use case, however, option 3 is fantastic. Instead of even depending on the actual audio, we could use Youtube's auto-generated subtitles in SRT format. This gives us both timestamp related information and the actual content in the video, in an easy-to-parse, compact format. The use of timestamps will be explained later. We collected all the content from the subtitles, and used ChatGPT to generate a summary. We then used Whisper to generate audio from the text summary, to eventually layer over the generated short video.
