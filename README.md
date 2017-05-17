Liform
======

Liform is a library for serializing Symfony Forms into [JSON schema][1]. It can
be used along with [liform-react][2] or [json-editor][3], or any other form
generator based in json-schema.

[1]: <http://json-schema.org/>

[2]: <https://github.com/Limenius/liform-react>

[3]: <https://github.com/jdorn/json-editor>

It is used by [LiformBundle][4] but can be used stand-alone.

[4]: <https://github.com/Limenius/LiformBundle>

It is very annoying to maintain backend forms that match forms in a client
technology, such as JavaScript. It is also annoying to maintain a documentation
of such forms. And error prone.

Liform generates a JSON schema representation, that serves as documentation and
can be used to document, validate your data and, if you want, to generate forms
using a generator.

[![Build Status][5]](https://travis-ci.org/Limenius/Liform) [![Latest Stable
Version][6]](https://packagist.org/packages/limenius/liform) [![Latest Unstable
Version][7]](https://packagist.org/packages/limenius/liform)
[![License][8]](https://packagist.org/packages/limenius/liform)

[5]: <https://travis-ci.org/Limenius/Liform.svg?branch=master>

[6]: <https://poser.pugx.org/limenius/liform/v/stable>

[7]: <https://poser.pugx.org/limenius/liform/v/unstable>

[8]: <https://poser.pugx.org/limenius/liform/license>

Installation
------------

Open a command console, enter your project directory and execute the following
command to download the latest stable version of this library:

    \$ composer require limenius/liform

This command requires you to have Composer installed globally, as explained in
the *installation chapter* of the Composer documentation.

>   Liform follows the PSR-4 convention names for its classes, which means you
>   can easily integrate `Liform` classes loading in your own autoloader.

\#\# Usage

Serializing a form into JSON Schema:

\`\`\`php use LimeniusLiformResolver; use LimeniusLiformLiform;

$resolver = new Resolver(); $resolver-\>setDefaultTransformers(); $liform = new
Liform($resolver);

$form = $this-\>createForm(CarType::class, $car, ['csrf_protection' => false]);
$schema = json\_encode($liform->transform($form)); \`\`\`

And `$schema` will contain a JSON Schema representation such as:

\`\`\`js {  
"title":null,    "properties":{  
"name":{  
"type":"string",  "title":"Name",  "propertyOrder":1   },   "color":{  
"type":"string",  "title":"Color",  "attr":{  
"placeholder":"444444"  },  "default":"444444",  "description":"3 hexadecimal
digits",  "propertyOrder":2   },   "drivers":{  
"type":"array",  "title":"hola",  "items":{  
"title":"Drivers", "properties":{  
"firstName":{  
"type":"string",   "propertyOrder":1    },    "familyName":{  
"type":"string",   "propertyOrder":2    } }, "required":[  
"firstName",    "familyName" ], "type":"object"  },  "propertyOrder":3   }    },
"required":[  
"name",   "drivers"    ] } \`\`\`

Using your own transformers
---------------------------

Liform works by inspecting recursively the form, finding (resolving) the right
transformer for every child and using that transformer to build the
corresponding slice of the json-schema. So, if you want to modify the way a
particular form type is transformed, you should set a transformer that matches a
type with that `block_prefix`.

To do so, you can use the `setTransformer` method of `Resolver`. In this case we
are reusing the StringTransformer, just making it set the widget property to
my\_widget, but you could use your very own transformer:

\`\`\`php

use LimeniusLiformResolver; use LimeniusLiformLiform; use
LimeniusLiformTransformer;

\$stringTransformer = new TransformerStringTransformer();

$resolver = new Resolver(); $resolver-\>setDefaultTransformers();
$resolver->setTransformer('my_block_prefix', $stringTransformer, 'my\_widget');
$liform = new Liform($resolver); \`\`\`

Serializing initial values
--------------------------

This library provides a normalizer to serialize a `FormView` (you can create one
with `$form->createView()`) into an array of initial values.

\`\`\`php use LimeniusLiformSerializerNormalizerFormViewNormalizer;

$encoders = array(new XmlEncoder(), new JsonEncoder()); $normalizers = array(new
FormViewNormalizer());

$serializer = new Serializer($normalizers, $encoders); $initialValues =
$serializer->normalize($form-\>createView()), \`\`\`

To obtain an array of initial values that match your json-schema.

\#\# Serializing errors

This library provides a normalizer to serialize forms with errors into an array.
This part was shameless taken from [FOSRestBundle][9]. Just do in your action:

[9]: <https://github.com/FriendsOfSymfony/FOSRestBundle/blob/master/Serializer/Normalizer/FormErrorNormalizer.php>

\`\`\`php use LimeniusLiformSerializerNormalizerFormErrorNormalizer;

$encoders = array(new XmlEncoder(), new JsonEncoder()); $normalizers = array(new
FormErrorNormalizer());

$serializer = new Serializer($normalizers, $encoders); $initialValues =
$serializer->normalize($form), \`\`\`

To obtain an array with the errors of your form. [liform-react][10], if you are
using it, can understand this format.

[10]: <https://github.com/Limenius/liform-react>

Information extracted to JSON-schema
------------------------------------

The goal of Liform is to extract as much data as possible from the form in order
to have a complete representation with validation and UI hints in the schema.
The options currently supported are.

Some of the data can be extracted from the usual form attributes, however, some
attributes will be provided using a special `liform` array that is passed to the
form options. To do so in a confortable way a [form extension][11] is provided.
See [AddLiformExtension.php][12]

[11]: <http://symfony.com/doc/current/form/create_form_type_extension.html>

[12]: <https://github.com/Limenius/Liform/blob/master/src/Limenius/Liform/Form/Extension/AddLiformExtension.php>

### Required

If the field is required (which is the default in Symfony), it will be reflected
in the schema.

\`\``php class DummyType extends AbstractType { public function
buildForm(FormBuilderInterface $builder, array $options) { $builder
->add('someText', Type\TextType::class); } } `\`\`

\`\``json {    "title":"dummy",    "type":"object",    "properties":{
"someText":{  "type":"string",  "title":"someText",  "propertyOrder":1   }    },
"required":[   "someText"    ] } `\`\`

### Widget

Some times it is not enough with the type to specify how this field should be
render, and you may want to specify that you would like to use a particular
widget.

If the attribute `widget` of `liform` is provided, as in the following code:

\`\``php class DummyType extends AbstractType { public function
buildForm(FormBuilderInterface $builder, array $options) { $builder
->add('someText', Type\TextType::class, [ 'liform' => [ 'widget' => 'my_widget'
] ]); } } `\`\`

The schema generated will have that `widget` option: \`\``json {
"title":"dummy",    "type":"object",    "properties":{   "someText":{
"type":"string",  "widget":"my_widget",  "title":"someText",  "propertyOrder":1
}    },    "required":[   "someText"    ] } `\`\`

### Label/Title

If you provide a `label`, it will be used as title in the schema.

\`\``php class DummyType extends AbstractType { public function
buildForm(FormBuilderInterface $builder, array $options) { $builder
->add('someText', Type\TextType::class, [ 'label' => 'Some text', ]); } } `\`\`

\`\``json {    "title":"dummy",    "type":"object",    "properties":{
"someText":{  "type":"string",  "title":"Some text",  "propertyOrder":1   }
},    "required":[   "someText"    ] } `\`\`

### Placeholder/Default

If the attribute `placeholder` of `attr` is provided, as in the following code:

\`\``php class DummyType extends AbstractType { public function
buildForm(FormBuilderInterface $builder, array $options) { $builder
->add('someText', Type\TextType::class, [ 'attr' => [ 'placeholder' => 'I am a
placeholder', ], ]); } } `\`\`

It will be extracted as the `default` option. Note that, in addition, everything
provided to `attr` will be preserved as well.

\`\``json {    "title":"dummy",    "type":"object",    "properties":{
"someText":{  "type":"string",  "title":"someText",  "attr":{ "placeholder":"I
am a placeholder"  },  "default":"I am a placeholder",  "propertyOrder":1   }
},    "required":[   "someText"    ] } `\`\`

### Pattern

If the attribute `pattern` of `attr` is provided, as in the following code:

\`\`\`php class DummyType extends AbstractType { public function
buildForm(FormBuilderInterface $builder, array $options) { \$builder
-\>add('someText', TypeTextType::class, [ 'attr' =\> [ 'pattern' =\> '.{5,}', ],
]); } }

\`\`` It will be extracted as the `pattern` option, so it can be used for
validation. Note that, in addition, everything provided to `attr\` will be
preserved as well.

\`\``json {    "title":"dummy",    "type":"object",    "properties":{
"someText":{  "type":"string",  "title":"someText",  "attr":{ "pattern":".{5,}"
},  "pattern":".{5,}",  "propertyOrder":1   }    },    "required":[   "someText"
] } `\`\`

### Description

If the attribute `description` of `liform` is provided, as in the following
code, it will be extracted in the schema:

\`\``php class DummyType extends AbstractType { public function
buildForm(FormBuilderInterface $builder, array $options) { $builder
->add('someText', Type\TextType::class, [ 'label' => 'Some text', 'liform' => [
'description' => 'This is a help message', ] ]); } } `\`\`

\`\``json {    "title":"dummy",    "type":"object",    "properties":{
"someText":{  "type":"string",  "title":"Some text",  "description":"This is a
help message",  "propertyOrder":1   }    },    "required":[   "someText"    ] }
`\`\`

License
-------

This library is under the MIT license. See the complete license in the file:

    LICENSE.md

Acknoledgements
---------------

The technique for transforming forms using resolvers and reducers is inspired on
[Symfony Console Form][13]

[13]: <https://github.com/matthiasnoback/symfony-console-form>

 
