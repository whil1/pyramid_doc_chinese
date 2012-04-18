.. index::
   single: application configuration

.. _configuration_narr:

应用程序配置 
=========================

众所周知，“配置”一词就是进行一些对应用程序会产生影响的操作。例如我们耳熟能
详的".ini"文件能在应用程序启动的时候被加载解析出里面的参数，这就是“配置”。
如果我们能稍微跳出一些过去的思维定势，那么就不难理解，其实*代码*也是一种配置。
Pyramid像许多其它的WEB应用平台一样，属于是一个*框架*，它会调用你所写的代码来
完成某些工作（这与*库*的意义刚好相反，*库*只是单纯地被你所写的代码调用。）
我们将所写的代码放入 :app:`Pyramid` 中，就像是在为它写配置文档一样；说白了，
其实就是让 :app:`Pyramid` 去主动调用我们所写的代码。

There are two ways to configure a :app:`Pyramid` application:
:term:`imperative configuration` and :term:`declarative configuration`.  Both
are described below.

.. index::
   single: imperative configuration

.. _imperative_configuration:

Imperative Configuration
------------------------

"Imperative configuration" just means configuration done by Python
statements, one after the next.  Here's one of the simplest :app:`Pyramid`
applications, configured imperatively:

.. code-block:: python
   :linenos:

   from wsgiref.simple_server import make_server
   from pyramid.config import Configurator
   from pyramid.response import Response

   def hello_world(request):
       return Response('Hello world!')

   if __name__ == '__main__':
       config = Configurator()
       config.add_view(hello_world)
       app = config.make_wsgi_app()
       server = make_server('0.0.0.0', 8080, app)
       server.serve_forever()

We won't talk much about what this application does yet.  Just note that the
"configuration' statements take place underneath the ``if __name__ ==
'__main__':`` stanza in the form of method calls on a :term:`Configurator`
object (e.g. ``config.add_view(...)``).  These statements take place one
after the other, and are executed in order, so the full power of Python,
including conditionals, can be employed in this mode of configuration.

.. index::
   single: view_config
   single: configuration decoration
   single: code scanning

.. _decorations_and_code_scanning:

Declarative Configuration
-------------------------

It's sometimes painful to have all configuration done by imperative code,
because often the code for a single application may live in many files.  If
the configuration is centralized in one place, you'll need to have at least
two files open at once to see the "big picture": the file that represents the
configuration, and the file that contains the implementation objects
referenced by the configuration.  To avoid this, :app:`Pyramid` allows you to
insert :term:`configuration decoration` statements very close to code that is
referred to by the declaration itself.  For example:

.. code-block:: python
   :linenos:

   from pyramid.response import Response
   from pyramid.view import view_config

   @view_config(name='hello', request_method='GET')
   def hello(request):
       return Response('Hello')

The mere existence of configuration decoration doesn't cause any
configuration registration to be performed.  Before it has any effect on the
configuration of a :app:`Pyramid` application, a configuration decoration
within application code must be found through a process known as a
:term:`scan`.

For example, the :class:`pyramid.view.view_config` decorator in the code
example above adds an attribute to the ``hello`` function, making it
available for a :term:`scan` to find it later.

A :term:`scan` of a :term:`module` or a :term:`package` and its subpackages
for decorations happens when the :meth:`pyramid.config.Configurator.scan`
method is invoked: scanning implies searching for configuration declarations
in a package and its subpackages.  For example:

.. topic:: Starting A Scan

   .. code-block:: python
      :linenos:

      from wsgiref.simple_server import make_server
      from pyramid.config import Configurator
      from pyramid.response import Response
      from pyramid.view import view_config
     
      @view_config()
      def hello(request):
          return Response('Hello')

      if __name__ == '__main__':
          from pyramid.config import Configurator
          config = Configurator()
          config.scan()
          app = config.make_wsgi_app()
          server = make_server('0.0.0.0', 8080, app)
          server.serve_forever()

The scanning machinery imports each module and subpackage in a package or
module recursively, looking for special attributes attached to objects
defined within a module.  These special attributes are typically attached to
code via the use of a :term:`decorator`.  For example, the
:class:`~pyramid.view.view_config` decorator can be attached to a function or
instance method.

Once scanning is invoked, and :term:`configuration decoration` is found by
the scanner, a set of calls are made to a :term:`Configurator` on your
behalf: these calls replace the need to add imperative configuration
statements that don't live near the code being configured.

The combination of :term:`configuration decoration` and the invocation of a
:term:`scan` is collectively known as :term:`declarative configuration`.

In the example above, the scanner translates the arguments to
:class:`~pyramid.view.view_config` into a call to the
:meth:`pyramid.config.Configurator.add_view` method, effectively:

.. ignore-next-block
.. code-block:: python
   :linenos:

   config.add_view(hello)

Summary
-------

There are two ways to configure a :app:`Pyramid` application: declaratively
and imperatively.  You can choose the mode you're most comfortable with; both
are completely equivalent.  Examples in this documentation will use both
modes interchangeably.
