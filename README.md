# symfony-info
In this repository, i store some easily accessible pointers on how to get going on a symfony project.

## initialising project

`$ symfony new <name> --version="7.2.x" --webapp`

OR

```
$ composer create-project symfony/skeleton:"7.2.x" <name>
$ cd <name>
$ composer require webapp
```

## starting server

`$ symfony server:start`

## make controller

`$ symfony console make:controller`
this asks for a controller name, just enter the name, and it will create a <Name>Controller in the controller folder
and a <name> folder in the templates folder. This is where twig files related to the controller are stored

### controller functions

if you wish to create a new page related to a controller, you must make a new public function in the controller. generally
these functions look like:

```php
#[Route('<route>', name: '<unique_name>')]
public function <name>(): Response
{
  ...
  return $this->render('<controller>/<template_name>.html.twig', [
    <twig_variable> => <value>,
    ]
  );
}
```
where:
- route is the desired route of the page 
- unique_name is the identifier of the route used in other places when working with the controller
- name is the name of the function, usually something related to CRUD
- controller is the name of the controller
- template_name is the name of the template used when rendering the route
- twig_variable and value are inside of a associative array where the twig_variable can be used in the template to where it is substituted by value in the rendered content

## databases

to connect to a database, you must alter the .env file and set the DATABASE_URL variable to the correct url of the database you wish to connect to

after correctly setting this url up, you can create the database by entering
`$ symfony console doctrine:database:create `

if done correctly, you will get a message that the database was created successfully

## make entity

you can make entities in your database by entering

`$ symfony console make:entity <name>`

which will start a dialog where symfony will ask you for a property name, type, length, and nullability.

after entering the desired amount of properties, you must migrate the entity by running

`$ symfony console make:migration`

if done correctly this should result in no errors

## Reading data

If you wish to interact with the data from an entity, you must do so via the controller. This is done by first adding the following line to the use statements in the controller file

```php
use App\Repository\ProductRepository;
```

this allows us to use methods related to the entity such as searching the table for entries, entering new data, and deleting unwanted data.

You can now add an argument to a function where you desire to work with an entity

```php
public function <name>(<entity_name>Repository $repository): Response { ... }
```

now you can use methods related to the entity in the function

examples are

- `findAll()` which returns all entites
- `find(<id>)` which returns a match based on the id provided
- `findBy([<key> => <value>])` which returns a match based on the column (key) and value provided (sql: WHERE <key> = <value>)
- `findOneBy([<key> => <value>])` which returns the first record found with the value provided in the column "<key>" (sql: WHERE <key> = <value> LIMIT 1) 

### Dumping data

If you wish to dump data from a variable, you can use the dump function and pass the variable as an argument
this puts a nicely formatted block on top of the rendered content

## Displaying data in Twig

To display data, you have to set it to a variable in the associative array in the return render statement in the controller.

As an example, if you put `'name' => 'John',` in the associative array, you can now use the variable `name` in your twig template.

### Twig crash course
- `{{ variable }}` puts the value of the variable in the render
- `{% statement %}` calls a twig related function, this is often `{% block <blockname> %}{% endblock %}` which allows the user to define a block that can be filled in later in a template that "extends" the "base template"
- `{% for <x> in <variable> %}...{% endfor%}` allows you to loop over <variable> if it is an array, where x is the value of each index passed
- `{{ path('<unique_name>', {'<route_variable>': <value>}) }}` is handy to make a link to a certain path, and allows for the setting of route variables 

## Route Variables

If you wish to have a route to for instance an individual product, you can achieve this by using route variables.

This is done as such:
```php
#[Route('/product/{id}', name: 'product_show')]
public function show($id): Response { ... }
```

The code above will now look at the route and use anything after /product/ as the "id" variable in the rest of the function. To restrict the types of values in the route variable, you may use regex enclosed by < > inside of the curly braces.
If we change the code above to

```php
#[Route('/product/{id<\d+>}', name: 'product_show')]
public function show($id): Response { ... }
```

the route will only accept digits as valid values, any other value will result in a 404 error.

## Exception throwing

If a user enters a value they are not supposed to see, you should probably make sure that they get a 404 page. This can be done by doing a check in code, and if something is wrong throwing an error as such:

```php
throw $this->createNotFoundException('<Message>');
```

### dev and prod settings

BY DEFAULT symfony starts off a project in dev mode, this allows you to see handy debug information when an error is thrown, however, if you wish to see how it would look if a client opened the site, you can change the APP_ENV
setting in the .env file from `dev` to `prod` which will run the site in production mode

## Shorthand of the show function above

To save on time and typing, you can use the following to achieve the same as using the ProductRepository, and find function:

```php
...
use App\Entity\<Entity>;

...

#[Route('/<entity>/{id}'), name: 'entity_show']
public function show(Product $product): Response { ... }

```

## Forms

To create a form, you must run the following:

`$ symfony console make:form`

This starts a dialog asking you for the name of the form class, and the entity to which this form is related to.

To use this form, you must first import it with `use App\Form\<Entity>Type;`

Now in the function in which you wish to use the form, you first create the form and assign it to a variable:
```php
$form = $this->createForm(<Entity>Type::class);
```

This can now be passed to the view by setting a render variable to form.

To render the form, in the twig file enter `{{ form(form) }}`, and it will render on site load, however by default this comes without a submit button

To add a submit button you must enter the form class file, and add it to the form builder:

```php
...
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
...
$builder
  ...
  ->add('save', SubmitType::class)
```

You can add a custom label to the button by adding the following:

```php
 ->add('save', SubmitType::class, [label => '<Custom Label>'])
```

This is now a full form, however clicking the button will not submit it. This must be implemented in the route function.

```php
...
use Symfony\Component\HttpFoundation\Request;
use Doctrine\ORM\EntityManagerInterface;
...
public function <name>(Request $request, EntityManagerInterface $em): Response
{
  $product = new Product;

  $form = $this->createForm(<Entity>Type::class, $product);
  $form->handleRequest($request);

  if ($form->isSubmitted()) {
    $manager->persist($product);
    $manager->flush();

    return $this->redirectToRoute('product_show');
  }
}

```

## Validation

Form validation by default is just done on the front end, however you can also add validation in the backend by doing the following.

Inside of the Entity file you wish to validate:

```php
...
use Symfony\Component\Validator\Constraints as Assert;

...

#[ORM\Column]
#[Assert\Not Blank] // Asserts that field must not be blank
#[Assert\Type('integer')] // Asserts that field must be an integer
#[Assert\Positive] // Asserts that field must be a positive integer
... // <- Entity Parameter
```

Now you can add

```php
... && $form->isValid() ...
```

to any submission check, and it will validate the field based on the assertions in the entity file.

## Flash messages

Flash messages are messages that appear only once, before disappearing after a reload, and can be added using the following inside a route function:

```php
$this->addFlash(
  '<type>',
  '<message>'
);
```

Then, in your twig template, you can add the following to set where the flash messages will be displayed:

```twig
{% for message in app.flashes %}
<div>
  {{ message }}
</div>
{% endfor %}
```

## Repeat Templates

When reusing the same snippets of template code, you may also use the include function built into twig. When this function is called
in a `{{ include(<path_to_template)>) }}` block, it will render that template in the current template. This is handy for forms, headers, etc.
The generally accepted file naming standard of these multiple use templates is to prefix it with an underscore. 

## Editing

To edit an entry in the database, you can use the same code as in the creation function, but without the persist call, as the record already exists in the database

## Deleting

To delete a record from the database, you use the remove method of the entity manager, and then flush the result.

```php
...
$em->remove($product);

$em->flush();
...
```

## CRUD generator

If you wish to skip over all of these steps, you can also make an entity, and afterwards use the crud generator built into the symfony console when in webapp mode (default if installed as instructed above)
This will generate the controller and templates for said entity, for each CRUD operation.

`$ symfony console make:crud`

This will start a dialog asking for the entity, controller name (default is usually good), and if you want to generate tests (not mandatory)

## Users

Symfony comes equipped with a generator for user classes. To generate one, simply run:

`$ symfony console make:user`

This will start a dialog, where you answer a few parameters of the User class.

This dialog will genreate an entity and repository, and you now must push a migration to the database with the new entity

### Registration form

Another handy thing built into symfony is the registration form generator.

`$ symfony console make:registration-form`

This starts a dialog asking you a couple of questions about the registration settings and such.
(Email verification requires a mail server, so i suggest saying no to email verification when making local projects)

This generator also creates the route `/register` where a user can register a new account

### Email restraint at registration

By default, there are no restraints on what you can enter in the email field in the registration form, this can be added by doing the following:

Inside the RegistrationFormType.php:
```php
...
use Symfony\Component\Form\Extension\Core\Type\EmailType;
...
use Symdony\Component\Valitdator\Constraints\Email;
...
$builder
  ->add('email', EmailType::class, [
    'constraints' => [
      new NotBlank([
        'message' => 'cannot be blank'
      ]),
      new Email ([
        'message => 'must be an email'
      ]),
    ]
  ])
```

### Login and Logout

Just as the register generator, symfony also has a login generator.

`$ symfony console make:security:form-login`

Once again this will start a dialogue where you have to answer to some config questions, and afterwards it will generate a login form.
This will create routes `/login/` and `/logout`, which allow for their respective actions

## User data

To use user data in twig, you use the app.user object, and use app.user.<parameter> to get the specific values of the user
