# Automaton [![Build Status](https://secure.travis-ci.org/IndigoUnited/automaton.png)](http://travis-ci.org/IndigoUnited/automaton)

Task automation tool built in JavaScript.


![Automaton](http://indigounited.github.com/automaton/img/stamp.png)


## Why?

You often find yourself needing to do some repetitive operation, and this is usually the time to quickly bake some ad-hoc script. Still, from project to project you find yourself needing to reuse some task you had already previously created.

Automaton eases this process, allowing you to quickly set up an `autofile`, which describes what you want to do, by means of an ordered list of tasks that need to run for the task as a whole to be complete.

A little detail that makes Automaton a powerful tool, is that every `autofile` you create can itself be used by another `autofile`, turning the first one into a single task (imagine boxes within boxes). If you are curious, you can take a look at the source code, and check for yourself that even the tasks that Automaton provides built-in are simple `autofiles`.


## Built-in tasks

`automaton` comes bundled with a few tasks to ease your own tasks.

### Filesystem

- [chmod](https://github.com/IU-Automaton/autofile-chmod): Change mode of files
- [cp](https://github.com/IU-Automaton/autofile-cp): Copy files and directories
- [mv](https://github.com/IU-Automaton/autofile-mv): Move files and directories
- [mkdir](https://github.com/IU-Automaton/autofile-mkdir): Make directories recursively
- [rm](https://github.com/IU-Automaton/autofile-rm): Remove several files or directories
- [symlink](https://github.com/IU-Automaton/autofile-symlink): Create symlink


### Scaffolding

Scaffolding tasks help you perform some typical tasks, like appending, replacing, and others, to placeholders in a template file. Any text file can be a template. These tasks will look for a `{{placeholder_name}}` inside the file, and perform the operation on it.

- [scaffolding-append](https://github.com/IU-Automaton/autofile-scaffolding-append): Append something to a placeholder in a file
- [scaffolding-replace](https://github.com/IU-Automaton/autofile-scaffolding-replace): Replace the placeholder with something
- [scaffolding-close](https://github.com/IU-Automaton/autofile-scaffolding-close): Close the placeholder (effectively removing the placeholder)
- [scaffolding-file-rename](https://github.com/IU-Automaton/autofile-scaffolding-file-rename): Rename files by replacing placeholders found in their names

### Miscellaneous

- [run](https://github.com/IU-Automaton/autofile-run): Run a shell command
- [init](https://github.com/IU-Automaton/autofile-init.git): Initialise an empty autofile
- uglify (soon)
- minify (soon)
- concat (soon)


## Installing

You can simply install Automaton through NPM, by running `npm install -g automaton`. This will install Automaton globally, and you will be able to execute `automaton` in your terminal.

If you only plan to use `automaton` programmatically, you can just install it locally.


## Creating a task

An automaton task is a simple object, describing what the task will do.

### Simple task

For illustration purposes, here's a simple `autofile` that just creates a folder and copies a file into it:

```js
module.exports = function (task) {
    task
    .id('my-task')
    .name('My task')
    .do('mkdir', {
        description: 'Create the project root folder',
        options: {
            dirs: ['some_dir']
        }
    })
    .do('cp', {
        description: 'Copy some file',
        options: {
            files: {
                'some_file': 'some_dir/dest_file'
            }
        }
    });
};
```

### More complete task


To illustrate most of the capabilities of `automaton`, here's a complete `autofile` with comments along the file:

```js
module.exports = function (task) {
    // Everything bellow is optional
    task
    // Allows to reference this task by id
    .id('example_task')
    // A user friendly name
    .name('Example task')
    // The task author
    .author('IndigoUnited')
    // The task description
    .description('My example task')

    // Options definition: name, description, default value
    .option('dir1', 'The name of the folder')
    .option('dir2', 'Another folder name', 'automaton')
    .option('run_all', 'Run everything', false)
    .option('debug', 'Enable/disable', false)

    // Function to run before everything
    .setup(function (options, ctx, next) {
        // You can change existing options
        options.dir2 = options.dir2 + '_indigo';

        // And even define additional options; in this case we're defining a 'dir3' option,
        // which will be used by one of the subtasks.
        options.dir3 = 'united';

        next();
    })

    // Function to run afterwards
    // Note that this function will run, even if some of the substask failed
    .teardown(function (options, ctx, next) {
        // do something afterwards (e.g.: cleanup something)
        next();
    })

    // A list of subtasks that will run
    .do('mkdir', {
        description: 'Create the root and second folder',
        options: {
            // the option below
            // will have its placeholders replaced by
            // the value that it receives.
            dirs: ['{{dir1}}/{{dir2}}']
        }
    })
    .do('mkdir', {
        // Description messages can be generated according to the options
        // by using a string with placeholders or a function.
        description: function (opt) {
            return 'Creating ' + opt.dir1 + '/' + opt.dir2 + '/' + opt.dir3
        },
        options: {
            dirs: ['{{dir1}}/{{dir2}}/{{dir3}}']
        },
        // This 'on' attributes allows you to enable/disable a subtask just by
        // setting it to a falsy value.
        // In this case, we even used a placeholder, allowing us tto skip this subtask
        // depending on the run_all option.
        // Of course, you have can set it to something like 'false'.
        on: '{{run_all}}',
        // This 'fatal' attribute allows to bypass tasks that fail, just by setting
        // it to a falsy value.
        // In this case, we even used a placeholder, allowing us to skip this subtask depending
        // on the debug option.
        fatal: '{{debug}}',
        // This 'mute' attribute allows you to completely mute log calls made inside
        // this task as well its subtasks.
        // In this case, we used 'false' but it could have been a placeholder.
        mute: false
    })
    // If you find yourself looking for something a bit more custom,
    // you can just provide a function as the task.
    // More details about inline functions below in the "Inline functions" section.
    .do(function (options, ctx, next) {
        // 'options' is a list of the options provided to the task.

        // ctx.log gives you access to the Logger.
        // The Logger should be used to perform any logging information, and is
        // preferred to any console.* methods, as this gives additional control.
        // More information on ctx in the "Inline Functions" section.
        ctx.log.writeln('I can do whatever I want', 'even works with multiple args');

        // When the task is done, you just call next()
        // not like the MTV show, though…
        // (- -')
        next();
    });
};
```

Note that placeholders can be escaped with backslashes:
`'\\{\\{dir1\\}\\}'`

### Inline functions

If you find yourself trying to do something that is not supported by the existing tasks, you can just provide a function, instead of the task name, and it will be used as the task.

This task will receive 3 arguments, an options object (the options that were provided to the subtask), a context object (more on this later), and a callback that must be called once the subtask is over, giving you full flexibility, since your function can do whatever you like.

The second argument, the context, is used to provide you with a tool belt that will aid you while developing `automaton` tasks. It currently provides you a `log` object, which is an instance of [Logger](https://github.com/IndigoUnited/automaton/blob/master/lib/Logger.js), and can be used to handle logging. Using the `automaton` logger is preferred to using the traditional `console.*` methods, since it gives additional control over logging to whoever is running the task.

The `Logger` provides the following methods:

- Normal logging: `write()`, `writeln()`
- Information logging: `info()`, `infoln()`
- Warnings logging: `warn()`, `warnln()`
- Error logging: `error()`, `errorln()`
- Success logging: `success()`, `successln()`
- Debug logging: `debug()`, `debugln()` (These will only be outputted when in debug mode)

The *ln* variants of each method output a new line (`\n`) in the end. Note that these methods work just like your typical `console.*` methods, so you can pass multiple arguments, and they will all get logged.

Here's an example usage:

```js
var inspect = require('util').inspect;

module.exports = function (task) {
    task
    .id('bogus')
    .do(function (options, ctx, next) {
        ctx.log.writeln(
            'hello,',
            'here\'s the process',
            inspect(process)
        );

        next();
    });
};
```

### Grunt tasks

You are able to run `grunt` tasks in `automaton`. It's actually very simple:

```js
module.exports = function (task) {
    task
    .id('bogus')
    .do('mincss', {
        grunt: true,
        options: {
            files: {
                'path/to/output.css': 'path/to/input.css'
            }
        }
    });
};
```

By default, `automaton` autoload tasks located in `tasks/` and npm tasks (that start with grunt-).
If your task lives in a different folder, you can specify it in the `grunt` config. Other grunt options like
`force` and `verbose` can also be specified:

```js
module.exports = function (task) {
    task
    .id('bogus')
    .do('some-grunt-task', {
        grunt: {
            tasks: ['lib/tasks/'],                // array of folders to load tasks from
            force: true,
            verbose: true
            // other grunt config goes here
        },
        options: {
            some: 'option'
        }
    });
};
```

### Loading tasks

Once you start building your own tasks, you will probably find yourself wanting to use some custom task within another task you're working on. In order to do this, you have a few options, depending on how you are using automaton.

If you are using `automaton` in the CLI, you have the `--task-dir`. This tells automaton to load all the tasks in that folder, making them available to you, just like the built-in tasks.

If you are using `automaton` programmatically, you have a bigger range of possibilities:

1. Run `automaton.loadTasks(/some/folder/with/tasks)`. This is the equivalent to what you would to in the CLI, with the exception that you can call this multiple times, loading multiple folders.

2. `require()` the task yourself, just like you would with any `NodeJS` module, and then call `automaton.addTask(your_task)`. Just like `loadTasks()`, this will make the task you just added available on `automaton` as if it is a built-in task.

3. `require()` the task in the task that depends on it, and use it directly in the subtask list, where you would typically put a task name, or an inline function.


## Usage

### CLI

All you need to use the CLI can be found by executing `automaton -h`. This will show you how to use `automaton`, and any of the loaded tasks.

In order to run an `autofile`, you simply run `automaton`. This will look for `autofile.js` in the current working dir. Instead, you can also run `automaton some_dir/my_autofile.js`, enabling you to specify what `autofile` you want to run.

### Node.js

`automaton` can also be used programmatically as a node module. Here's a quick example of its usage:

```js
var automaton = require('automaton').create(/*options go here*/);

// Since autofiles are node modules themselves,
// you can just require them
// Note that you could have instead declared
// the module inline, in JSON.
var myTask = require('my_autofile');

// Note that we're running a task that you have loaded using node's
// require, and passing it as the first argument of the run() function.
// Instead, you can load the task using loadTask(), and then simply
// pass its id (a string), as the first argument of run. You can find an
// example of this below, in the Logging section.
automaton.run(myTask, { 'some_option': 'that is handy' }, function (err) {
    if (err) {
        console.log('Something went wrong: ' + err.message);
    } else {
        console.log('All done!');
    }
});
```

#### Logging

`automaton.run()` returns a readable stream that is used for outputting log information. The depth of this log can be controlled by a `verbosity` option, provided upon instantiation.

There are also a few other options. Here's a full list:

- **verbosity:** Controls the depth of the log. Remember the box in a box analogy? This controls how many boxes deep you want to go, regarding logging information.
  - **0:** no logging
  - **1:** 1 level deep (default)
  - ***n*:** *n* levels deep
  - **-1:** show all levels
- **debug:** If you want to receive debug logging messages. `false` by default.
- **color:** If you want the logging information to contain colors. `true` by default.

Here's an example of the usage of the stream, with custom options:

```js
var automaton = require('automaton').create({
    verbosity: 3, // show me 3 level deep logging info
    debug: true,  // show me debug logging
    color: false  // disable colors
});

// run some task
automaton
.run('run', { cmd: 'echo SUCCESS!' })
.pipe(process.stdout);
```

As you can see, we've tweaked all the `automaton` options, and even piped the logging information to `STDOUT`. What you do exactly with the stream, it's completely up to you.


## Acknowledgements

Should be noted that this tool was inspired by already existing tools, and you should definitely take a look at them before deciding what is the right tool for the job at hand:

- [Initializr](http://www.initializr.com/), by [Jonathan Verrecchia](https://twitter.com/verekia)
- [Gruntjs](http://gruntjs.com/), by [Ben Alman](https://twitter.com/cowboy)

To these guys, a big thanks for their work.

Also, big thanks to [Ricardo Pereira](http://designer-freelancer.com/), for his awesome work with the mascot.

## Contributing

Should be noted that Automaton is an open source project, and also work in progress. Feel free to contribute to the project, either with questions, ideas, or solutions. Don't forget to check out the issues page, as there are some improvements planned.

Thanks, and happy automation!


## License

Released under the [MIT License](http://www.opensource.org/licenses/mit-license.php).
