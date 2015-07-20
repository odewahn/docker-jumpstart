# Docker Jumpstart, by Andrew Odewahn

This is the source for the Docker Jumpstart, a short guide by Andrew Odewahn from [O'Reilly Media](http://www.oreilly.com/) to help you get up and running with [Docker](https://www.docker.com/).  Its table of contents is:

* [Introduction](public/introduction.md).  A basic intro about what Docker is and isn't.
* [boot2docker](public/boot2docker.md).  How to get Docker running on Mac or Windows.
* [Images: Layered filesystems](public/docker-images.md).  A brief explanation of images and layers.
* [Containers: Running instances of an Image](public/containers.md).  Using `docker run` to turn a static image into a running container.
* [Creating your own Docker Image](public/example.md).  Putting everything together to make your own images.
* [Building images with Dockerfiles](public/building-images-with-dockerfiles.md).  Dockerfiles, a lightweight IA tool to describe an image's contents.  
* [Sharing images on Docker Hub](public/dockerhub.md).  Finding and sharing images with the world.
* [Additional Resources](public/additional-resources.md).  Where to learn more once you've mastered the basics of Docker.

Topics I'd love to cover in more depth are in the [wishlist](wishlist/) directory. Most of this stuff is just a place to keep links and research will (OK, might) someday lead to a more in-depth coverage.



## Contributing

This book is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License](http://creativecommons.org/licenses/by-nc/4.0/).  I encourage you to fork my work and make your own improvements under the terms of the license. If you have any changes you want to send back our way, please make a regular pull request via Github. If the authors like your changes, they may integrate them into the official repo and give you a credit. If you just have an issue to report, please use the regular Github issue system.


## Running the site locally

The site uses [harpjs](http://harpjs.com/) to build a site and [Atlas](https://atlas.oreilly.com/) to build the PDF, mobi, and epub.  To build the site:

* [Install harp](http://harpjs.com/docs/quick-start).  
* Clone the repo
* Run `harp server --port 4567`

When you want to build the site, clone the repo down again into another directory, create a gh-pages branch, and then run `harp compile . _new directory_`
