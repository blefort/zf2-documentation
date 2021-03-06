.. _zend.code.generator.introduction:

Introduction
============

``Zend\Code\Generator`` provides facilities to generate arbitrary code using an object-oriented interface, both to
create new code as well as to update existing code. While the current implementation is limited to generating *PHP*
code, you can easily extend the base class in order to provide code generation for other tasks: JavaScript,
configuration files, apache vhosts, etc.

.. _zend.code.generator.introduction.theory:

Theory of Operation
-------------------

In the most typical use case, you will simply instantiate a code generator class and either pass it the appropriate
configuration or configure it after instantiation. To generate the code, you will simply echo the object or call
its ``generate()`` method.

.. code-block:: php
   :linenos:

   // Passing configuration to the constructor:
   $file = new Zend\Code\Generator\FileGenerator(array(
       'classes' => array(
           new Zend\Code\Generator\ClassGenerator(array(
               'name'    => 'World',
               'methods' => array(
                   new Zend\Code\Generator\MethodGenerator(array(
                       'name' => 'hello',
                       'body' => 'echo \'Hello world!\';',
                   )),
               ),
           )),
       )
   ));

   // Configuring after instantiation
   $method = new Zend\Code\Generator\MethodGenerator();
   $method->setName('hello')
          ->setBody('echo \'Hello world!\';');

   $class = new Zend\Code\Generator\ClassGenerator();
   $class->setName('World')
         ->setMethod($method);

   $file = new Zend\Code\Generator\FileGenerator();
   $file->setClass($class);

   // Render the generated file
   echo $file;

   // or write it to a file:
   file_put_contents('World.php', $file->generate());

Both of the above samples will render the same result:

.. code-block:: php
   :linenos:

   <?php

   class World
   {

       public function hello()
       {
           echo 'Hello world!';
       }

   }

Another common use case is to update existing code -- for instance, to add a method to a class. In such a case, you
must first inspect the existing code using reflection, and then add your new method. ``Zend\Code\Generator`` makes
this trivially simple, by leveraging :ref:`Zend\Code\Reflection <zend.code.reflection>`.

As an example, let's say we've saved the above to the file ``World.php``, and have already included it. We could
then do the following:

.. code-block:: php
   :linenos:

   $class = Zend\Code\Generator\ClassGenerator::fromReflection(
       new Zend\Code\Reflection\ClassReflection('World')
   );

   $method = new Zend\Code\Generator\MethodGenerator();
   $method->setName('mrMcFeeley')
          ->setBody('echo \'Hello, Mr. McFeeley!\';');
   $class->setMethod($method);

   $file = new Zend\Code\Generator\FileGenerator();
   $file->setClass($class);

   // Render the generated file
   echo $file;

   // Or, better yet, write it back to the original file:
   file_put_contents('World.php', $file->generate());

The resulting class file will now look like this:

.. code-block:: php
   :linenos:

   <?php

   class World
   {

       public function hello()
       {
           echo 'Hello world!';
       }

       public function mrMcFeeley()
       {
           echo 'Hellow Mr. McFeeley!';
       }

   }


