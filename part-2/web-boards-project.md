# Проект Web Board 

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/11/a-complete-beginners-guide-to-django-part-2.html

Не знаю, как вы, но я узнаю намного больше через практические примеры и сниппеты кода. Мне сложно понять принцип, когда в примерах написано `Class A`, `Class B` или классическое `foo(bar)`. И я бы не хотел делать это в этом руководстве.

So, before we get into the fun part, playing with models, views, and everything. Let’s just take a moment and discuss very briefly about this project we are going to develop.

If you already have experience with Web development and feel it’s too much detail, you can just skim through the pictures to have an idea what we are going to build and then jump to the **Models** section of this tutorial.

But if you are new to Web development, I highly suggest that you keep reading. It will give you some good insights on modeling and design of Web applications. Web development, and software development in general, is not just about coding.

![Rocket Science](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_Rocket_Science.png)

##### Use Case Diagram

Our project is a discussion board (a forum). The whole idea is to maintain several **boards**, which will behave like categories. Then, inside a specific board, a user can start a new discussion by creating a new **topic**. In this topic, other users can engage in the discussion posting replies.

We will need to find a way to differentiate a regular user from an admin user because only the admins are supposed to create new boards. Below, an overview of our main use cases and the role of each type of user:

![Use Case Diagram](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/use-case-diagram.png)

<small>Figure 1: Use case diagram of the core functionalities offered by the Web Board</small>

##### Class Diagram

From the Use Case Diagram, we can start thinking concerning the **entities** of our project. The entities are the models we will create, and it’s very closely related to the data our Django app will process.

For us to be able to implement the use cases described in the previous section, we will need to implement at least the following models: **Board**, **Topic**, **Post**, and **User**.

![Basic Class Diagram](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/basic-class-diagram.png)

<small>Figure 2: Draft of the class diagram of the Web Board</small>

It’s also important to take the time to think about how do models will relate to each other. What the solid lines are telling us is that, in a **Topic**, we will need to have a field to identify which **Board** it belongs to. Similarly, the **Post** will need a field to represent which **Topic** it belongs so that we can list in the discussions only **Posts** created within a specific **Topic**. Finally, we will need fields in both the **Topic** to know who started the discussion and in the **Post** so we can identify who is posting the reply.

We could also have an association with the **Board** and the **User** model, so we could identify who created a given **Board**. But this information is not relevant for the application. There are other ways to track this information, you will see later on.

Now that we have the basic class representation, we have to think what kind of information each of those models will carry. This sort of thing can get complicated very easily. So try to focus on the important bits. The information that you need to start the development. Later on, we can improve the model using **migrations**, which you will see in great detail in the next tutorial.

But for now, this would be a basic representation of our models’ fields:

![Class Diagram](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram.png)

<small>Figure 3: Class diagram emphasizing the relationship between the classes (models)</small>

This class diagram has the emphasis on the relationship between the models. Those lines and arrows will eventually be translated into fields later on.

For the **Board** model, we will start with two fields: **name** and **description**. The **name** field has to be unique, so to avoid duplicated board names. The **description** is just to give a hint of what the board is all about.

The **Topic** model will be composed of four fields: **subject**, **last update** date which will be used to define the topics ordering, **topic starter** to identify the **User** who started the **Topic**, and a field called **board** to define which **Board** a specific **Topic** belongs to.

The **Post** model will have a **message** field, which will be used to store the text of the post replies, a **created at** date and time field mainly used to order the **Posts** within a **Topic**, an **updated at** date and time field to inform the **Users** when and if a given **Post** has been edited. Like the date and time fields, we will also have to reference the **User** model: **created by** and **updated by**.

Finally, the **User** model. In the class diagram, I only mentioned the fields **username**, **password**, **email** and **is superuser** flag because that’s pretty much all we are going to use for now. It’s important to note that we won’t need to create a **User** model because Django already comes with a built-in **User** model inside the **contrib** package. We are going to use it.

Regarding the multiplicity in the class diagram (the numbers `1`, `0..*`, etc.), here’s how you read it:

![Class Diagram Board and Topic Association](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-board-topic.png) A **Topic** must be associated with exactly one (`1`) **Board** (which means it cannot be null), and a **Board** may be associated with many **Topics** or none (`0..*`). Which means a **Board** may exist without a single **Topic**.

![Class Diagram Topic and Post Association](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-topic-post.png) A **Topic** should have at least one **Post** (the starter **Post**), and it may also have many **Posts** (`1..*`). A **Post** must be associated with one, and only one **Topic** (`1`).

![Class Diagram Topic and User Association](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-topic-user.png) A **Topic** must have one, and only one **User** associated with: the topic starter **User** (`1`). And a **User** may have many or none **Topics** (`0..*`).

![Class Diagram Post and User Association](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-post-user.png) A **Post** must have one, and only one **User** associated with: **created by** (`1`). A **User** may have many or none **Posts** (`0..*`). The second association between **Post** and **User** is a _direct association_ (see the arrow at the end of the line), meaning we are interested only in one side of the relationship which is what **User** has edited a given **Post**. It will be translated into the **updated by** field. The multiplicity says `0..1`, meaning the **updated by** field may be null (the **Post** wasn’t edited) and at most may be associated with only one **User**.

Another way to draw this class diagram is emphasizing the fields rather than in the relationship between the models:

<div id="figure-4">

![Class Diagram Attributes](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-attributes.png)

<small>Figure 4: Class diagram emphasizing the attributes (fields) of the classes (models)</small>

</div>

The representation above is equivalent to the previous one, and it’s also closer to what we are going to design using the Django Models API. In this representation, we can see more clearly that in the **Post** model the associations **topic**, **created by**, and **updated by** became model fields. Another interesting thing to note is that in the **Topic** model we have now an _operation_ (a class method) named **posts()**. We are going to achieve this by implementing a reverse relationship, where Django will automatically execute a query in the database to return a list of all the **Posts** that belongs to a specific **Topic**.

Alright, enough UML for now! To draw the diagrams presented in this section I used the [StarUML](http://staruml.io) tool.

##### Wireframes

After spending some time designing the application models, I like to create some wireframes to define what needs to be done and also to have a clear picture of where we are going.

![Wireframes Comic](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_Wireframes.png)

Then based on the wireframes we can gain a deeper understanding of the entities involved in the application.

First thing, we need to show all the boards in the homepage:

<div id="figure-5">

![Wireframe Boards](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/wireframe-boards.png)

<small>Figure 5: Boards project wireframe homepage listing all the available boards.</small>

</div>

If the user clicks on a link, say in the Django board, it should list all the topics:

<div id="figure-6">

![Wireframe Topics](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/wireframe-topics.png)

<small>Figure 6: Boards project wireframe listing all topics in the Django board.</small>

</div>

Here we have two main paths: either the user clicks on the “new topic” button to create a new topic, or the user clicks on a topic to see or engage in a discussion.

The “new topic” screen:

<div id="figure-7">

![Wireframe New Topic](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/wireframe-new-topic.png)

<small>Figure 7: New topic screen</small>

</div>

Now the topic screen, displaying the posts and discussions:

<div id="figure-8">

![Wireframe Posts](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/wireframe-posts.png)

<small>Figure 8: Topic posts listing screen</small>

</div>

If the user clicks on the reply button, they will see the screen below, with a summary of the posts in reverse order (newest first):

<div id="figure-9">

![Wireframe Reply](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/wireframe-reply.png)

<small>Figure 9: Reply topic screen</small>

</div>

To draw your wireframes you can use the [draw.io](https://draw.io) service, it’s free.

* * *