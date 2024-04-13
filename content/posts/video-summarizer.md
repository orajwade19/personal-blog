
# Lessons from building an automated video summary generator 
Last week I went to the MongoDB Generative AI Hackathon in NYC! It was a really fun experience, and I learned a lot about leveraging state-of-the-art AI to build incredible things. At the same time, I also learned about some limitations of the current machine learning space, and got some experience with nailing down the appropriate "scope" of a project meant to be built in 6 hours.
 Let's start with the idea.
Imagine you have a long soccer video, and you want to get just the highlights from the video. What do you do? Here's the basic idea.

> Summarize the video --> Generate summary audio --> Get the right clips from the video, corresponding to the summary --> Stitch it all together for the final video

 Now for generating the summary, we have a couple of options.
1. Use a model trained on videos to directly get the description of the video
2. Get the image frames for the video, use an image captioning model to describe all the images, and fuse together the description to "understand" the video 
3. Use the commentary in the video to understand the most important things in it, and use the corresponding video segments.
4. use 2 and 3 together.

Option 1 is not feasible, especially for a 6-hour hackathon. Video AI is still very much a work-in-progress, and finding models that suit our purpose while being easy to deploy is hard. Potential benefits of this approach would have been the reduction of data pre-processing requirements and its applicability to a wide range of videos without the need for specific adaptations. It could work just as well for both soccer clips and animal planet clips. If I was Google, option 1 would be tempting.

Coming to option 2, an immediate benefit that stood out to us was that this approach would work for "silent" videos as well, since it depends only on the images in the video. 
