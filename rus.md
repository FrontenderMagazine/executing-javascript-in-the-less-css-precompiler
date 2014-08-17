<article id="blog-post">
#Исполнение JavaScript кода в LESS CSS препроцессоре 

На днях, когда я модернизировал свое LiveReload приложение, некоторые из моих
LESS миксинов начали сбоить. В процессе отладки, я обнаружил, что некоторые из
миксинов исполняли JavaScript код непосредственно в LESS CSS разметке. Я
понятия не имел, что это было даже возможно в LESS CSS; и так, как и со всеми
вещами в JavaScript, я начал экспериментировать. У меня нет конкретных идей о
том, для чего я могу это использовать; но, вот, что я нашел. Пожалуйста,
заметьте, что это все лиш гипотеза, так как нет никакой  окументации (только
то что я смог найти) на использование JavaScript в LESS CSS.

JavaScript может быть оценен только как часть задания. Под этим я
подразумеваю, что вы не можете просто выполнить JavaScript в LESS CSS.  место
этого, вы должны выполнить JavaScript, а затем присвоить результат LESS
переменной или свойству CSS. Пример:

    @foo: `javascript` ;

Для обозначения Javascript кода, вам необходимо обернуть свой код в обратные
кавычки. Когда вы это сделаете,  LESS будет вычислять выражение и возвращать
результат в виде строки. Он делает это, «как есть»; смысл, он оставляет все
аши кавычки на месте. Если вы хотите удалить окружающие кавычки, вы можете
добавить тильду:

    @foo: ~`"surrounding quotes will be removed"`

Тем не менее, вот что я придумал. Я определил объект «API», который содержит
мои пользовательские функции (UDFs). Затем я вставил этот объект API в
лобальную контейнер, который позволяет «API» экземпляру ссылаться во все
другие JavaScript блоки:


    // Use the backtick character to run JavaScript directly in LESS CSS. We
    are using a
    // Function here because LESS calls (my theory) .toString() on the 
    function and stores
    // the return value. It seems that only Functions returns their "source 
    code" when
    // .toString() is called, which allows us to reuse the JavaScript in 
    other JavaScript
    // code block instances.
    @api: `function(){
     
        // Set up the API that we want to expose in other JavaScript contexts.
        var api = {
     
            // This is just me testing some JavaScript stuff.
            getContent: function() {
     
                return( "This is yo' content!" );
     
            },
     
     
            // When executing a JavaScript expression, you can only execute 
    one expression
            // at a time (ie, no semi-colons). Unless, you are in a function.
    The point of
            // the run() function is to allow the calling context a way to 
    enter a function
            // context and get access to the API at the same time. The API 
    is injected as the
            // only argument to the given callback.
            // --
            // NOTE: The callback MUST RETURN A VALUE so that LESS can get 
    the value of it.
            run: function( callback ) {
     
                return( callback( api ) );
    
            }
    
        };
     
    
        // Return the public API. Since this JavaScript expression is return
    the parent
        // Function, it will have to invoked in a different JavaScript 
    context to actually
        // get access to the API.
        return( api );
    
    }` ;
    
    
    // I assign the API the global namespace, "api". This could have been done
    in the
    // previous JavaScript code block; but, I kind of liked the idea of 
    breaking it out into
    // its own rsponsability.
    // --
    // NOTE: I am using a self-invoking function here to help ensure that 
    "this" points to
    // the global context and not to the context of the evaluation (if its 
    different).
    @apiGlobalInjector: `(function() {
    
        // Inject the API and store it in the global object.
        this.api = (@{api})();
     
        // The JavaScript HAS TO RETURN something so LESS CSS can assigne the
    variable value.
        return( "Injected in global" );
    
    })()` ;
    
     
    // ---------------------------------------------------------- //
    // ---------------------------------------------------------- //
    
    
    h1 {
        content: `api.getContent()` ;
    }
    
    h2 {
        content: `api.run(function( api ) {
        return( api.getContent().toUpperCase() );
        })` ;
    }

Когда вы исполняете JavaScript блок, LESS  CSS-у кажется, не нравится с точка
с запятой; если же вы не в функции. Таким образом, я создал метод .run(),
который был просто способом создать функцию, которая самостоятельно  ставляет
API. Таким образом, вы можете запустить несколько строк кода и возвратить их
значение.


Во всяком случае, при запуске компиляция вышеуказанный файл, LESS CSS дает
следующий результат:

    h1 {
        content: "This is yo' content!";
    }
    h2 {
        content: "THIS IS YO' CONTENT!";
    }

Как я уже говорил, я не совсем уверен, как я бы хотел  это использовать; но,
то определенно щекочет воображение. Есть ли что нибудь иное, это вдохновило
меня, чтобы узнать больше о все LESS  CSS в качестве фреймворка.</article>