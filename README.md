# React + Netty (w/o Spring Boot) practice

Build a simple one-page app to practice React.js with Spring Boot

</br>
</br>

## Directory Structure
<img width="319" alt="Screenshot 2023-05-29 at 6 17 19 PM" src="https://github.com/davidoh-0808/1Page-React-Netty-Toy/assets/75977587/4e295d62-cce3-454f-80b9-ec1a65ece36f">

</br>

## React
### What to build?
![image](https://github.com/davidoh-0808/1Page-React-Netty-Toy/assets/75977587/b63fd446-64f7-4fae-b9b8-3c0580afdaeb)
</br>

  Start react-app by
  ```
    npx create-react-app react-app
  ```
  
  In the react app, Index.js contains React DOM which will gather all the components, css and js from App.js and render it to index.html.
  
     const root = ReactDOM.createRoot(document.getElementById('root'));
     root.render(
      <React.StrictMode>
        <App />
      </React.StrictMode>
    );
    
  
  App.js will is like the gateway for all the component js and css files

  ```
    import './App.css';
    import {Component} from "react";

    class App extends Component {

      render() {
        return (
            <div className="App">
              <h1>
                Movies
              </h1>
            </div>
        )
      }
    }

    export default App;
   ```

  This 1-page react app will have two compoents: </br>
  1) MovieList </br>
 
  ```
    import React, { Component } from "react";
    import axios from "axios";
    import './MovieList.css';

    class MovieList extends Component {

    /*
        MovieList Component has a constructor which inherits react props.
        Bind functions like genreLabel and toMovie to this props.
     */
    constructor(props) {
        super(props);
        this.toMovie = this.toMovie.bind(this);
        this.genreLabel = this.genreLabel.bind(this);
    }

    genreLabel(g) {
        return "?";
    }

    toMovie(m) {    // TODO : Fetch  1. movies  2. genres via axios

        var g = "?";
        for (var i = 0; i < this.props.genres.length; i++) {
            if (this.props.genres[i].value == m.genre) {
                g = this.props.genres[i].label;
                break;
            }
        }

        return(
            <tbody key={m.id}>
                <tr>
                    <td>{m.title}</td><td>{g}</td>
                </tr>
            </tbody>
        )
    }

    render() {
        return(
            <table className="movie-list">
                <tbody>
                    <tr>
                        <th>Title</th>
                        <th>Genre</th>
                    </tr>
                </tbody>

                {/* TODO : Fetch  1. movies  2. genres via axios */}
                {this.props.movies.map(this.toMovie(m))}
            </table>
        )
    }
  ```

  and 2) MovieForm (still work in progress..!)
  
  ```
    import React, { Component } from "react";
    import axios from "axios";
    import './MovieForm.css';

    //new
    var xhr;

    class MovieForm extends Component {

        /*
            MovieForm Component has a constructor which inherits react props.
            1. Bind functions like handleChangeTitle and handleChangeGenre to the react props.
            2. Also, the react props provides a set of state which keeps track of the current states
                of variables like "title" and "genre".  Set the state with the variables of the form

            Use
         */
        constructor(props) {
            super(props);
            this.state = { title: '', genre: 'action' };

        }

    }

  export default MovieForm;
  ```

</br>
</br>


## Netty
### Configuration
  
          
  
  #### Why Use Netty?
  ```  
  HTTP servers are application-layer implementations of the HTTP protocol (OSI Layer 7), 
so relatively high up in the internet stack. If you’re developing a REST API, 
you’re developing on top of an API that provides this implementation for you. 
By contrast, Netty doesn’t necessarily structure communication, provide session management, or even offer security like TLS. 
This is great if you’re building a super low-level networking application; 
  
    Pros:
  Netty allows you to write your own network protocol tailored to your specific requirements, 
optimizing the traffic flow for your specific situation, without the unnecessary overhead of something like HTTP or FTP.
  
    Cons:
  Perhaps not the best choice if you’re building a REST service.

  Fortunately, the Netty API also provides some helper classes and functions 
that will allow us to easily integrate a higher level protocol like HTTP. 

  ```
</br>

  1) Configure Netty App Server
  
  ```

public class NettyAppServer {

    // The Netty Server PORT must be different from Spring Boot
    private static final int HTTP_PORT = 8888;

    public void run() throws Exception {
        // Create the multithreaded event loops for the server
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            // A helper class that simplifies server configuration
            ServerBootstrap httpBootstrap = new ServerBootstrap();

            // Configure the server
            httpBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)

                    /*
                        ** netty channel handler is registered here **
                        netty channel handler is like a gateway for all "controllers" as in typical REST API.  << I assume?

                        The actual "controllers" are registered in this channel handler
                     */
                    .childHandler(new NettyServerInitializer())

                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true);
            // Bind and start to accept incoming connections.
            ChannelFuture httpChannel = httpBootstrap.bind(HTTP_PORT).sync();
            // Wait until server socket is closed
            httpChannel.channel().closeFuture().sync();
        }
        finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }

}
  ```
</br>

  2) Configure Netty Server Initializer
  
  ```
public class NettyServerInitializer extends ChannelInitializer<Channel> {

    @Override
    protected void initChannel(Channel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();

        pipeline.addLast(new HttpServerCodec());

        /* used to gather the received data (max integer byte value) in ChannelHandlerContext - the ctx -
           which can later be used inside handlers ? */
        pipeline.addLast(new HttpObjectAggregator(Integer.MAX_VALUE));

        // the actual request handlers (similar to controllers in REST API) are registered here ?
        pipeline.addLast(new NettyChannelHandlerMovie());
    }

}
  ```
</br>

  3) Lastly.. configure Netty Channel Handler for the movie service </br>
  (sort of like a Controller in a typical REST API) </br>
  (it's still a work-in-progress..!)
  
  ```
  public class NettyChannelHandlerMovie extends SimpleChannelInboundHandler<FullHttpRequest> {

    /* ** SPRING IOC ISSUE **
    private MovieController movieController;

    NettyChannelHandlerMovie() {
        this.movieController = new MovieController(

                new MovieServiceImpl( new MovieRepository( ** SPRING IOC ISSUE ** ) )
        );
    }
    */

    private List<Movie> movieList = new ArrayList<>();

    // so manually injecting data.. for time sake
    NettyChannelHandlerMovie() {
        movieList.add( Movie.builder().id(1).genre("action").title("John Wick").build() );
        movieList.add( Movie.builder().id(2).genre("action").title("The Fast And Furious").build() );
        movieList.add( Movie.builder().id(3).genre("sci-fi").title("Interstellar").build() );
        movieList.add( Movie.builder().id(4).genre("animation").title("Your Name Is").build() );
        movieList.add( Movie.builder().id(5).genre("drama").title("The Godfather").build() );
        movieList.add( Movie.builder().id(6).genre("comedy").title("Central Intelligence").build() );
    }

    // test this Netty endpoint by sending Postman request via "GET or POST http://localhost:8888/movies"
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, FullHttpRequest msg) throws Exception {

        // ChannelHandlerContext and FullHttpRequest together contain comprehensive data, such as header info, request url, message and more, as found in lower level packet information.  Unpacking these data can facilitate creating a customized server.. Very facsinating..!

        String reqMethod = msg.method().name(); // << either POST or GET
        String reqUri = msg.uri(); // << either POST /movies or GET /movies

        // (1) POST (INSERT) Movie
        if (reqMethod.equals("POST") && reqUri.equals("movies")) {

            /* ** SPRING IOC ISSUE **
            MovieController.saveMovie(Movie movie) */

            /* configure POST
               read byteBuf from Netty Channel and convert it into Java Object Movie*/
            ByteBuf byteBuf = msg.content();

            byte[] byteArray = new byte[byteBuf.readableBytes()];
            byteBuf.getBytes(byteBuf.readerIndex(), byteArray);

            ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(byteArray);
            ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);

            Object byteBufToJavaObject = objectInputStream.readObject();

            objectInputStream.close();
            byteArrayInputStream.close();

            // TODO : CHECK .. not sure????
            Movie newMovie = (Movie) byteBufToJavaObject;
            movieList.add(newMovie);

        }

        // (2) GET (SELECT) Movie
        if (reqMethod.equals("GET") && reqUri.equals("movies")) {

            /* ** SPRING IOC ISSUE **
            invoke controller .getAllMovies() */

            // configure Netty response
            // << insert the actual movie data here ? TODO: CHECK
            ByteBuf headerContent = Unpooled.copiedBuffer( new Gson().toJson(movieList), CharsetUtil.UTF_8);

            FullHttpResponse resp = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, headerContent);
            resp.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/html");

            // insert the actual movie data here ? TODO: CHECK
            resp.headers().set(HttpHeaderNames.CONTENT_LENGTH, headerContent.readableBytes());

            // send the message via Netty Pipeline and remove it afterward
            ctx.write(resp);
            ctx.flush();

        }


    }
}
  ```

</br>
</br>
