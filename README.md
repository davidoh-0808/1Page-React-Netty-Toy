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
  
    npx create-react-app react-app

  In the react app, Index.js contains React DOM which will gather all the components, css and js from App.js and render it to index.html.
  
     const root = ReactDOM.createRoot(document.getElementById('root'));
     root.render(
      <React.StrictMode>
        <App />
      </React.StrictMode>
    );
    
  
  App.js will is like the gateway for all the component js and css files

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

  This 1-page react app will have two compoents: </br>
  1) MovieList </br>
 
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

  and 2) MovieForm (still work in progress..!)
  
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


</br>
</br>


## Netty
### Configuration
  code
  
    asdfasdf
</br>

  code
  
    asdfasdf
</br>

  code
  
    asdfasdf
</br>

  code
  
    asdfasdf


</br>
</br>
