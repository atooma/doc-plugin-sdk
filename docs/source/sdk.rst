.. _sdk:

Getting Started
=======================================

Atooma Software Development Kit allows you to write your own modules for Atooma app.

Introduction
---------------------------------------

Execute following steps:

1. Get registered and follow the instructions for downloading the SDK zip file, unzip it and open the tools folder.

2. Launch our script to create a skeleton for your first plugin:

.. code-block:: bash

  /create_atooma_plugin PluginName /path/to/plugin/directory

Now you can start working on your plugin! Using Eclipse you can go to

.. code-block:: bash

  File -> New -> Project -> Android Project from existing Code

Browse the folder you created.

The script will set the packagename as ``com.atooma.plugin.pluginname``; if you want to change it or if you do not want to use the script you will still need to use a packagename that begins with ``com.atooma.plugin.*``.

Modules
---------------------------------------

Modules are foundamental of Atooma, They rappresent tipically a specific type of feature and contains **Triggers**, **Condition Checkers** and **Performers**. To make a **Module** you have to:

* Write the Constructor

* Implements ``registerComponents()`` where you can register **Tirggers**, **Condition Checkers** and **Performers** and implements ``defineUI()``

Each components (**Modules**, **Triggers**, **Condition Checkers** and **Performers**) that implements ``defineUI()`` can call ``setIcon()`` and ``setTitle()`` giving as parameters the int id recource of a string or drawable from its project.

Triggers
---------------------------------------

Triggers are components that are responsible to forward a notification when a specific event occours. They are the IF part of a rule. Each trigger can specify a list of input parameters and output variables; parameters and variables must be declared in each trigger class. There are 3 types of trigger:

* ``Trigger`` - the standard one, ``onInvoke`` method is called when the rule is activated

* ``IntentBasedTrigger`` - ``onReceive()`` method is called when a specified intent is called

* ``AlarmBasedTrigger`` - ``onTimeout()`` method is called at a specific moment set by the developer

When a trigger wants to notify Atooma about the rule to be triggered, it have call the ``trigger()`` method.

Condition Checkers
---------------------------------------

Condition Checkers are components that check if a condition is ``true`` or ``false`` in the moment they are invoked and returns a Boolean value. Condition Checkers must implement ``onInvoke()`` method and must be accompanied by an identical Trigger (same UI settings, same id, etc.) Condition Checkers are used by Atooma when a user make a rule with two or more component in the IF sections.

Performers
---------------------------------------

Performers are components invoked by Atooma when a rule is triggered. Performers must implements ``onInvoke()`` method.

Create Your First Module
---------------------------------------

To create a Module, just extend ``Module`` class from the SDK. Create the constructor and call ``super()`` like this:

.. code-block:: java
  :linenos:

  public TEST(Context context, String id, int version) {
    super(context, id, version);
  }

You need to implement two methods, ``registerComponents`` and ``defineUI``. The first method, ``registerComponents``, allows you to register Triggers, ConditionCheckers and Performers. Call ``registerTriggers()`` or ``registerConditionChecker()`` or ``registerPerformer()`` to register a component like this:

.. code-block:: java
  :linenos:
  
  public void registerComponents() {
    registerTrigger(new TR_Trigger(getContext(), "TRN", 1));
    registerConditionChecker(new CC_ConditionChecker(getContext(), "CC", 1));
    registerPerformer(new PE_Performer(getContext(), "PE", 1));
  }

The second method, ``defineUI``, allows you to define a UI for your plugins, specifying icon resources (normal and pressed icon) and a title resource.

.. code-block:: java
  :linenos:

  public void defineUI() {
    setIcon(R.drawable.icon_normal, R.drawable.icon_pressed);
    setTitle(R.string.module_name);
  }

If you want to display your plugin in Connections section you just have to:

* Call ``setAuthenticated(boolean, String)`` inside of ``defineAuth()`` method in ``Module`` class. The boolean value indicates if the user is authenticated or not and the String value is the username

* Have at least one ``PLUGIN`` parameter defined with a valid ``Activity``
In ``clearCredentials`` you can log out the user or clear the login info. For example:

.. code-block:: java
  :linenos:

  @Override
  public void defineAuth() {
    SharedPreferences sp = getContext().getSharedPreferences("Prefs", Context.MODE_MULTI_PROCESS);
    String authText = sp.getString("AutenticatedText", "");
    if (authText.length() > 0) {
      setAuthenticated(true, authText);
    } else {
      setAuthenticated(false, "");
    }
  }

  @Override
  public void clearCredentials() {
    SharedPreferences sp = getContext().getSharedPreferences("Prefs", Context.MODE_MULTI_PROCESS);
    sp.edit().clear().commit();
  }

We suggest to use ``Context.MODE_MULTI_PROCESS`` for ``SharedPreferences``.
In your ``Activity`` you have obviously to save the credentials in some way, for example:

.. code-block:: java
  :linenos:

  Intent intent = new Intent();
  SharedPreferences sp = getSharedPreferences("Prefs", 0);
  sp.edit().putString("AutenticatedText", string).commit();
  intent.putExtra(AtoomaParams.ACTIVITY_RESULT_KEY, string);
  setResult(RESULT_OK, intent);
  finish();

Create A Trigger
---------------------------------------

To create a **Trigger**, just extend ``Trigger`` class from the SDK. Create the constructor and call ``super()``. You need to implement two methods, ``defineUI()`` and ``onInvoke()``. In ``defineUI()`` you can set icon and title as you did with the Module. The method ``onInvoke`` here is called when the rule is activated; here you can insert your code and if you actually want to trigger your rule you can call ``trigger()``. If you want to use ``AlarmBasedTrigger`` you have to implement ``getScheduleInfo()`` like this

.. code-block:: java
  :linenos:
  
  public Schedule getScheduleInfo() {
    long now = System.currentTimeMillis();
    long triggerAtTime = now + 10000L;
    Schedule schedule = new Schedule.Builder().exact(true).triggerAtTime(triggerAtTime).build();
    return schedule;
  }

and then ``onTimeout`` will be called at the selected time. If you want to use ``IntentBasedTrigger`` you have to implement ``getIntentFilter()`` like this:

.. code-block:: java
  :linenos:

  public String getIntentFilter() {
    return Intent.ACTION_BATTERY_CHANGED;
  }

and then onReceive will be called when the intent will be fired.

Create A Condition Checker
---------------------------------------

To create a ConditionChecker, just extend ``ConditionChecker`` class from the SDK. Create the constructor and call ``super()``. You need to implement two methods, ``defineUI`` and ``onInvoke``. In ``defineUI()`` you can set icon and title as you did with the Module and Triggers. The method ``onInvoke`` here is called when the trigger selected by the user befoure this Condition Checker will be triggered, if the condition is satisfied you can return ``true``, otherwise return ``false``.

Create A Performer
---------------------------------------

To create a Performer, just extend ``Performer`` class from the SDK. Create the constructor and call ``super()``. You need to implement two methods, ``defineUI`` and ``onInvoke``. In ``defineUI()`` you can set icon and title as you did with the Module, Triggers and Condition Checker. The method ``onInvoke`` here is called when the rule made by the user has triggered, and here you can do the code you want to execute.

Set Parameters And Variables
---------------------------------------

For each component, Triggers, ConditionCheckers and Performers you can set parameters and variables with the same code. Parameters are the values that the user inserts while he is creating the rule and they will be avaiable in onInvoke method (or ``onReceive`` or ``onTimeout``). You can set the parameters like this:

.. code-block:: java
  :linenos:
  
  public void declareParameters() {
    addParameter(R.string.parameter_name, R.string.parameter_ifnull, "NAME", "STRING", true, null);
  }

Where ``R.string.parameter_name`` is the title of the parameter, ``R.string.parameter_ifnull`` is the title of the parameter in the editor if the user won’t set a parameter, ``NAME`` is the id of the parameter, ``STRING`` the type, true indicate if the parameter is required or not to the user, and null is the string that indicate the path of an activity on your package that can be launched when the user edit the rule. Then in ``onInvoke`` (for example) if you want to use the parameters you can get it in this way:

.. code-block:: java
  :linenos:

  String myParameters = (String) parameters.get("NAME");

Variables are the values that Triggers and Performers can pass in output. Triggers can pass some values to a Performers and Performers to other Performers; The data can be passed from variabiles to parameters also through different plugin components or different Atooma hardcoded components. You can set the variables like this:

.. code-block:: java
  :linenos:
  
  @Override
  public void declareVariables() {
    addVariable(R.string.parameter_name, "NAME", "STRING");
  }

Values allowed:

* ``STRING``
* ``BOOLEAN``
* ``NUMBER`` (Double)
* ``PLUGIN`` (it’s like ``STRING``, but you can use it with your own activity)
* ``URI``

With ``PLUGIN`` value you can use your own ``Activity`` in order to take a ``String`` as parameter. You have to use ``addParameter`` like this:

.. code-block:: java
  :linenos:

  addParameter(R.string.parameter_name, R.string.parameter_ifnull, 
    "NAME", "PLUGIN", true, "com.atooma.plugin.test.MainActivity");

And in the ``MainActivity`` you have to set the result:

.. code-block:: java
  :linenos:

  Intent intent = new Intent();
  intent.putExtra(AtoomaParams.ACTIVITY_RESULT_KEY, string);
  setResult(RESULT_OK, intent);
  finish();

Then in ``onInvoke`` you can get the parameter:

.. code-block:: java
  :linenos:

  String myParameters = (String) parameters.get("NAME");

Versioning
---------------------------------------

Each component (Module, Trigger, ConditionChecker and Performer) can be created with a number which indicates the component's version. Versioning is very important in Atooma to avoid crashes and has to be incremented everytime a component or a new parameter/variable has been added. For example if you add a new Trigger to the Module you have to increment the Module version and indicate that the new Trigger is avaiable only through the new Module version.

Example before:

.. code-block:: java
  :linenos:

  public static final String MODULE_ID = "MODULETEST";
  public static final int MODULE_VERSION = 1;

  public TEST(Context context, String id, int version) {
    super(context, id, version);
  }

  @Override
  public void registerComponents() {
    registerTrigger(new TR_Trigger(getContext(), "TRONE", 1));
  }

After:

.. code-block:: java
  :linenos:

  public static final String MODULE_ID = "MODULETEST";
  public static final int MODULE_VERSION = 2;

  public TEST(Context context, String id, int version) {
    super(context, id, version);
  }

  @Override
  public void registerComponents() {
    registerTrigger(new TR_Trigger(getContext(), "TRONE", 1));
    registerTrigger(new TR_Trigger(getContext(), "TRTWO", 2));
  }
  
If I added a new parameter to the trigger ``TRONE`` I would have to increment them too.


