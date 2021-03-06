.. _using_procfiles:

Using Procfiles
===============

As described in :ref:`what_are_procfiles`, Procfiles are simple text files that
describe the components required to run an applications. This document describes
some of the more advances features of Honcho and the Procfile ecosystem.

Syntax
------

The basic syntax of a Procfile is described in `the Heroku Procfile
documentation
<https://devcenter.heroku.com/articles/procfile#declaring-process-types>`_. In
summary, a Procfile is a plain text file placed at the root of your applications
source tree that contains zero or more lines of the form::

    <process type>: <command>

The ``process type`` is a string which may contain alphanumerics and underscores
(``[A-Za-z0-9_]+``), and uniquely identifies one type of process which can be
run to form your application. For example: ``web``, ``worker``, or
``my_process_123``.

``command`` is a shell commandline which will be executed to spawn a process of
the specified type.

Environment files
-----------------

You can also create a ``.env`` file alongside your Procfile which contains
environment variables which will be available to all processes started by
Honcho::

    $ cat >.env <<EOF
    RACK_ENV=production
    ASSET_ROOT=https://myapp.s3.amazonaws.com/assets
    EOF

Typically, you should not commit your ``.env`` file to your version control
repository, but you might wish to create a ``.env.example`` so that others
checking out your code can see what environment variables your application uses.

For more on why you might want to use environment variables to configure your
application, see Heroku's article on `configuration variables`_ and The
Twelve-Factor App's `guidance on configuration`_.

.. _configuration variables: https://devcenter.heroku.com/articles/config-vars
.. _guidance on configuration: http://12factor.net/config

Using Honcho
------------

To see the command line arguments accepted by Honcho, run it with the ``--help``
option::

    $ honcho --help
    usage: honcho [-h] [-v] [-e ENV] [-d APP_ROOT] [-f PROCFILE]
                  {check,export,help,run,start} ...

    Manage Procfile-based applications

    optional arguments:
      -h, --help            show this help message and exit
      -v, --version         show program's version number and exit

    common arguments:
      -e ENV, --env ENV     Environment file[,file] (default: .env)
      -d APP_ROOT, --app-root APP_ROOT
                            Procfile directory (default: .)
      -f PROCFILE, --procfile PROCFILE
                            Procfile path (default: Procfile)

    tasks:
      {check,export,help,run,start}
        check               Validate your application's Procfile
        export              Export the application to another process management
                            format
        help                Describe available tasks or one specific task
        run                 Run a command using your application's environment
        start               Start the application (or a specific PROCESS)


You will notice that by default, Honcho will read a Procfile called
``Procfile`` from the current working directory, and will read environment from
a file called ``.env`` if one exists. You can override these options at the
command line if necessary. For example, if your application root is a level
above the current directory and your Procfile is called ``Procfile.dev``, you
could invoke Honcho thus::

    $ honcho -d .. -f Procfile.dev start
    16:14:49 web.1 | started with pid 1234
    ...

If you supply multiple comma-separated arguments to the ``-e`` option, Honcho will merge the environments provided by each of the files::

    $ echo 'ANIMAL_1=giraffe' >.env.one
    $ echo 'ANIMAL_2=elephant' >.env.two
    $ honcho -e .env.one,.env.two run sh -c 'env | grep -i animal'
    ANIMAL_1=giraffe
    ANIMAL_2=elephant
