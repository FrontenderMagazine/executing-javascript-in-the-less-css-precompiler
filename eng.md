<article id="blog-post">
The other day, when I upgraded my LiveReload app, some of my LESS mixins
started to break. During the debugging process, I discovered that some of the 
mixins were executing JavaScript code directly in the LESS CSS markup. I had no 
idea that this was even possible in LESS CSS; and so, as with all things 
JavaScript, I had to start experimenting. I have no concrete ideas on what I 
would use this for; but, here's what I found. Please note that this is all 
conjecture as there is zero documentation (that I could find) on the use of 
JavaScript in LESS CSS.

JavaScript can only be evaluated as part of an assignment. By this I mean that
you can't simply execute JavaScript in LESS CSS. Instead, you have to execute 
the JavaScript and then assign the result to a LESS variable or a CSS property. 
Example:



To denote JavaScript, you have to wrap your code in backticks. When you do this
, LESS will evaluate the expression and return the result as a String. It does 
so, "as is;" meaning, it leaves all your quotes in-place. If you want to remove 
the surrounding quotes, you can prepend the tilde operator:



*   @foo: ~\`"surrounding quotes will be removed"\`

That said, here's what I came up with. I defined an "API" object that holds my
user-defined functions (UDFs). I then inject this API object into the global 
container which allows the "api" instance to be referenced in all the other 
JavaScript code blocks:



*   // Use the backtick character to run JavaScript directly in LESS CSS. We
    are using a
   
*   // Function here because LESS calls (my theory) .toString() on the function
    and stores
   
*   // the return value. It seems that only Functions returns their "source
    code" when
   
*   // .toString() is called, which allows us to reuse the JavaScript in other
    JavaScript
   
*   // code block instances.
*   @api: `function(){
*      
    
*   // Set up the API that we want to expose in other JavaScript contexts.
*   var api = {
*      
    
*   // This is just me testing some JavaScript stuff.
*   getContent: function() {
*      
    
*   return( "This is yo' content!" );
*      
    
*   },
*      
    
*      
    
*   // When executing a JavaScript expression, you can only execute one
    expression
   
*   // at a time (ie, no semi-colons). Unless, you are in a function. The point
    of
   
*   // the run() function is to allow the calling context a way to enter a
    function
   
*   // context and get access to the API at the same time. The API is injected
    as the
   
*   // only argument to the given callback.
*   // --
*   // NOTE: The callback MUST RETURN A VALUE so that LESS can get the value of
    it.
   
*   run: function( callback ) {
*      
    
*   return( callback( api ) );
*      
    
*   }
*      
    
*   };
*      
    
*      
    
*   // Return the public API. Since this JavaScript expression is return the
    parent
   
*   // Function, it will have to invoked in a different JavaScript context to
    actually
   
*   // get access to the API.
*   return( api );
*      
    
*   }` ;
*      
    
*      
    
*   // I assign the API the global namespace, "api". This could have been done
    in the
   
*   // previous JavaScript code block; but, I kind of liked the idea of
    breaking it out into
   
*   // its own rsponsability.
*   // --
*   // NOTE: I am using a self-invoking function here to help ensure that "this
    " points to
   
*   // the global context and not to the context of the evaluation (if its
    different
    ).
*   @apiGlobalInjector: `(function() {
*      
    
*   // Inject the API and store it in the global object.
*   this.api = (@{api})();
*      
    
*   // The JavaScript HAS TO RETURN something so LESS CSS can assigne the
    variable value.
   
*   return( "Injected in global" );
*      
    
*   })()` ;
*      
    
*      
    
*   
   
*   
   
*      
    
*      
    
*   h1 {
*   content: \`api.getContent()\` ;
*   }
*      
    
*   h2 {
*   content: `api.run(function( api ) {
*   return( api.getContent().toUpperCase() );
*   })` ;
*   }

When you execute a JavaScript block, LESS CSS doesn't seem to like semi-colons
; unless you are in side of a function. As such, I created a .run() method, 
which was just a way to create a function that self-injects the API. This way, 
you could run multiple statements and return a value.

Anyway, when you run the compile the above file, LESS CSS produces the
following output:



*   h1 {
*   content: "This is yo' content!";
*   }
*   h2 {
*   content: "THIS IS YO' CONTENT!";
*   }

Like I said before, I am not exactly sure how I would use this yet; but, it
definitely tickles the imagination. If nothing else, it's inspired me to learn 
more about LESS CSS as a framework.</article>