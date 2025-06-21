# Carlos Moreno's Portfolio

[cmoreno.me](http://cmoreno.me) will hold my portfolio of various frontend and full stack projects.

There will be different subdomains with live sites for each project.

- [hello.cmoreno.me](http://hello.cmoreno.me): simple Express app serving "Hello
  World". There's also a basic chatbot app (from the Frontend Masters course
  [Full Stack for Frontend v2](https://frontendmasters.com/courses/archive/fullstack-v2/)).
- [book.cmoreno.me](http://book.cmoreno.me): small Express app for managing a
  list of books. Based on a tutorial article: [Building a REST API with Node and
  Express](https://stackabuse.com/building-a-rest-api-with-node-and-express/)

Eventually, I'll add styling and layout to the main site, and blocks
representing each project. For now, the main site is a list of links to each
project.

## Practice using Nginx as a load balancer

I found a video tutorial on configuring Nginx as a load balancer for multiple
application servers. While I don't foresee the need for Dockerized app instances
for small projects, it was a good use case for the tutorial, and I learned a lot
about Nginx configurations.

I didn't want to put my notes in a separate repo, so [they are in this
repo](nginx-configs/README.md).
