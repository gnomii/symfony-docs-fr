.. index::
   single: Form; Utiliser des formulaires virtuels

Comment utiliser l'Option de Champ de Formulaire ``Virtual`` 
============================================================

L'option de champ de formulaire ``virtual`` peut être très utile quand vous
avez des champs dupliqués dans différentes entités.

Par exemple, imaginez que vous ayez deux entités, une ``Company`` et une ``Customer``::

    // src/Acme/HelloBundle/Entity/Company.php
    namespace Acme\HelloBundle\Entity;

    class Company
    {
        private $name;
        private $website;

        private $address;
        private $zipcode;
        private $city;
        private $country;
    }

.. code-block:: php

    // src/Acme/HelloBundle/Entity/Customer.php
    namespace Acme\HelloBundle\Entity;

    class Customer
    {
        private $firstName;
        private $lastName;

        private $address;
        private $zipcode;
        private $city;
        private $country;
    }

Comme vous pouvez le voir, chaque entité partage quelques champs qui sont
identiques : ``address``, ``zipcode``, ``city``, ``country``.

Maintenant, vous voulez construire deux formulaires : un pour l'entité
``Company`` et un autre pour l'entité ``Customer``.

Commencez par créer un ``CompanyType`` et un ``CustomerType``::

    // src/Acme/HelloBundle/Form/Type/CompanyType.php
    namespace Acme\HelloBundle\Form\Type;

    use Symfony\Component\Form\FormBuilderInterface;

    class CompanyType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('name', 'text')
                ->add('website', 'text');
        }
    }

.. code-block:: php

    // src/Acme/HelloBundle/Form/Type/CustomerType.php
    namespace Acme\HelloBundle\Form\Type;

    use Symfony\Component\Form\FormBuilderInterface;
    use Symfony\Component\OptionsResolver\OptionsResolverInterface;

    class CustomerType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('firstName', 'text')
                ->add('lastName', 'text');
        }
    }

Maintenant, les quatre champs dupliqués doivent être gérés. Vous pouvez
voir ci-dessous un (simple) type de formulaire « location » (« lieu » en
français)::

    // src/Acme/HelloBundle/Form/Type/LocationType.php
    namespace Acme\HelloBundle\Form\Type;

    use Symfony\Component\Form\FormBuilderInterface;
    use Symfony\Component\OptionsResolver\OptionsResolverInterface;

    class LocationType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('address', 'textarea')
                ->add('zipcode', 'string')
                ->add('city', 'string')
                ->add('country', 'text');
        }

        public function getDefaultOptions(array $options)
        {
            return array(
                'virtual' => true,
            );
        }

        public function getName()
        {
            return 'location';
        }
    }

Vous n'avez *en fait* pas de champ « location » dans vos entités, donc vous
ne pouvez pas lier directement votre ``LocationType`` à votre ``CompanyType`` ou à votre
``CustomerType``. Mais vous voulez absolument avoir un type de formulaire dédié pour
gérer le lieu (rappelez-vous, DRY - Don't Repeat Yourself!).

L'option de champ de formulaire ``virtual`` est la solution.

Vous pouvez définir l'option ``'virtual' => true`` dans la méthode
``setDefaultOptions()`` de ``LocationType`` et commencer à l'utiliser directement 
dans les deux types de formulaires initiaux.

Voyez le résultat::

    // CompanyType
    public function buildForm(FormBuilderInterface $builder, array $options)
    {  
        $builder->add('foo', new LocationType(), array( 
            'data_class' => 'Acme\HelloBundle\Entity\Company'
        ));
    }

.. code-block:: php

    // CustomerType
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('bar', new LocationType(), array(
            'data_class' => 'Acme\HelloBundle\Entity\Customer'
        ));
    }

Avec l'option « virtual » définie à « false » (comportement par défaut),
le composant Form s'attend à ce que chaque objet sous-jacent ait une propriété
``foo`` (ou ``bar``) qui soit un objet ou un tableau contenant les quatre
champs du lieu. Bien sûr, vous n'avez pas cet objet/tableau dans vos
entités et vous ne le voulez pas.

Avec l'option « virtual » définie à « true », le composant Form ne s'occupe pas
de la propriété ``foo`` (ou ``bar``), et à la place « récupère » et « définit » (« gets »
et « sets » en anglais) les 4 champs du lieu directement sur l'objet sous-jacent.

.. note::

    Au lieu de définir l'option ``virtual`` dans le type ``LocationType``,
    vous pouvez (comme pour n'importe quelle autre option) aussi la passer
    comme une option sous forme de tableau en tant que troisième argument de
    ``$builder->add()``.
