---
layout: post
title: Machine Learning with JavaScript
---

During my last semester of college, a former professor of mine had recommended Andrew Hg's [Machine Learning](https://www.coursera.org/learn/machine-learning/) course to me. I didn't think much of it until a few months ago when I started at [Nashville Software School](http://nashvillesoftwareschool.com/). I enrolled in the class near the end of my first month at NSS, and I can say that I'm hooked on studying machine learning now.

As the end of the first half of Nashville Software School approached, we were asked to build a frontend capstone project demonstrating everything we have learned. I wanted to integrate what I've learned from the Coursera Machine Learning Course with the concepts taught during the frontend portion of the bootcamp. In the machine learning course, I had been exposed to neural networks and some of their applications (at the time of writing this post, I am still working through the course). In the bootcamp, we built [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) applications that used [Angular 1.x](https://angularjs.org/).

The goals for our frontend capstone were straight forward:

1. Create an application using Angular 1.x.

2. The application should connect to Firebase.

3. The application should have CRUD capabilities.

4. The application should have some form of user authentication.

5. OPTIONAL: The application should pull data from an external API.

I looked around at a few APIs to get an idea of what I wanted to build. As an avid music fan, I looked into the [Spotify Web API](https://developer.spotify.com/web-api/).

I'm a daily Spotify user. Friends and I had built grandiose Spotify playlists in the past. Our playlists usually fit a specific theme. However, we did have one friend who tried to add songs that did not fit with the current theme for the playlist we were working on at the time. There was a time that he tried to add [Take it Easy by the Eagles](https://www.youtube.com/watch?v=LfeNhwnO8hw) to a funky, disco playlist. The Eagles are a great band, but I'm sure anyone who has listened to them would agree that they aren't a disco group (Don't take my word for it, Wikipedia says otherwise, too).

<img src="https://imgur.com/olzgkt6.jpg" alt="The Eagles band" style="width:50%;display:block;margin:0 auto">

Continuing my search through the Spotify API, I found an interesting endpoint: [Get Audio Features](https://developer.spotify.com/web-api/get-audio-features/). An idea clicked in my head for my capstone project:

> Build an application that helps users make smart decisions when adding new songs to a playlist. A user should be able to ask the application, "Would this new song fit with this playlist?" The application would use neural networks to learn from a Spotify playlist by learning each song's audio features.

Spotify currently has the _opposite_ feature implemented in their desktop client. Spotify can recommend songs that fit with a playlist, but a user can't ask Spotify if a song would fit. This is where neural networks come into play.

<img src="https://i.imgur.com/3i8RH0r.png" alt="Spotify song recommendation feature" style="width:75%;display:block;margin:0 auto">

# What is a Neural Network?

_Machine learning and artificial intelligence experts, please forgive my attempt to describe a neural network in one sentence._

A [neural network](https://en.wikipedia.org/wiki/Artificial_neural_network) is an adaptive network of artifical neurons that can learn patterns from a set of data. This Youtube video covers the basics:

<iframe style="display:block;margin:0 auto" width="560" height="315" src="https://www.youtube.com/embed/DG5-UyRBQD4" frameborder="0" allowfullscreen></iframe>

The Spotify Web API's [audio features endpoint](https://developer.spotify.com/web-api/get-audio-features/) returns a lot of song metadata. For the sake of training the neural network with audio features, I only considered the features that were scaled from 0 to 1. This was to prevent [overfitting](https://en.wikipedia.org/wiki/Overfitting). Had I included all of the features returned from the endpoint, [certain features would have skewed the network](https://www.quora.com/What-is-regularization-in-machine-learning).

Here's the network structure used to learn a user's playlist:

<img src="{{ site.baseurl }}/images/BrainifyNetwork.svg" alt="Brainify Neural Network" style="height:40%;display:block;margin:0 auto">

# Neural Networks and the Frontend

I looked for neural network libraries to get started. JavaScript is not a top choice for machine learning experts, so it was challenging to find anything that wasn't an abandoned project (See this [Quora post on why JavaScript is not a popular choice for ML](https://www.quora.com/Why-doesnt-anyone-recommend-JavaScript-Node-js-as-a-language-for-machine-learning-or-data-analysis)). I came across [Synaptic.js](http://caza.la/synaptic/#/), an actively maintained open-source neural network library.

Using Synaptic.js, the application relies on a mere 13 lines of code to train a neural network. For each song in the playlist, the network is trained to output 1. For songs that are not in the playlist, the network is trained to output 0.

{% highlight javascript %}
/**
 * Trains the neural network with songs
 * @param  {Array<float>} playlistSongFeatures
 *     Song features for the playlist
 * @param  {Array<float>} wrongSongsFeatures
 *     Song features for nonplaylist songs
 * @return {void}
 */
function trainNetwork(playlistSongFeatures, wrongSongsFeatures) {
  const learningRate = 0.01;
  for (let i = 0; i < 40000; i++) {
    playlistSongFeatures.forEach((song) => {
      playlistNetwork.activate(song);
      playlistNetwork.propagate(learningRate, [1]);
    });

    wrongSongsFeatures.forEach((song) => {
      playlistNetwork.activate(song);
      playlistNetwork.propagate(learningRate, [0]);
    });
  }
}
{% endhighlight %}

This allows us to ask the neural network its opinion on whether or not a song fits with the playlist:

{% highlight javascript %}
function makePrediction(song) {
  return myNetwork.activate(song);
}
{% endhighlight %}

The biggest hurdle was figuring out how to find songs that are not in a playlist. The neural network can't only learn from the list of songs in a playlist. I'll refer to this list of songs that are not found in the playlist as the **negative sample**. Songs that do exist in the playlist will be refered to as the **positive sample**.

The positive sample is easy to figure out. It contains every song in the playlist. The negative sample isn't so easy to find. It could contain any song that is not in the playlist. A blues playlist could contain songs by Howlin' Wolf, Muddy Waters, and John Lee Hooker. But what about the negative sample? It could contain synth-pop, rock, jazz, Christmas songs, and virtually any other song from any genre that is not blues.

<img src="{{ site.baseurl }}/images/DataSample.svg" alt="Brainify Neural Network" style="width:60%;display:block;margin:0 auto">

The solution to this problem took some trial and error, but after playing with Spotify's API, I found a viable answer.

Spotify has data available for 126 genres. If I could find the list of genres present in a playlist, I could find the list of genres that are not present.

Spotify stores genre tags for each artist in its database (most of the time). If I looked up Metallica via Spotify's API, I would get back data classifying the group as "heavy metal," "metal," and "rock." I could then search the list of 126 genres and remove any genres present in the playlist. What's remaining is a list of genres that are not in the playlist.

Spotify allows users to retrieve a list of [song recommendations from a genre](https://developer.spotify.com/web-api/get-recommendations/). I saved the data of 20 song recommendations for each genre that Spotify has data available for (nearly 122 out of 126 genres). This allowed me to create a database filled with a small sample songs from virtually every genre that Spotify recognizes.

I found that the neural network trains the best when the ratio of positive to negative samples is close to 1:1, so the application takes a random sampling of size _n_, where _n_ is equal to the number of songs in the playlist, of songs from every other genre I have in my database. This corrected the imbalance between the positive and negative sample sizes.

<img src="{{ site.baseurl }}/images/DataSampleCorrect.svg" alt="Brainify Neural Network" style="width:60%;display:block;margin:0 auto">

Anecdotally, I have found that the resulting neural network accurately predicts whether a song should fit in a playlist. I have yet to test the neural network with a large sample of songs to determine the network's real accuracy.

Users can use the neural networks in the application to build smarter Spotify playlists. They can even test out a friend's playlist and send them song recommendations through the application.

Now, I need to get my friend to create an account in the application so I can show him that the Eagles don't belong in our funk playlist.

For more about the project, check out the [repo on Github](https://github.com/matthamil/Brainify).

For a great visual introduction to Machine Learning, check out this post by [R2D3](http://www.r2d3.us/visual-intro-to-machine-learning-part-1/).
