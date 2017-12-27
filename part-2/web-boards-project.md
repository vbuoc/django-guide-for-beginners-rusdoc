# Проект Web Board 

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/11/a-complete-beginners-guide-to-django-part-2.html

Не знаю, как вы, но я узнаю намного больше через практические примеры и сниппеты кода. Мне сложно понять принцип, когда в примерах написано `Class A`, `Class B` или классическое `foo(bar)`. И я бы не хотел делать это в этом руководстве.

Теперь, прежде чем мы приступим к веселой части, играя с моделями, представлениями и всем остальным, давайте поверхностно о проекте, который мы будем разрабатывать.

Если вы уже имеете опыт Web-разработки и считаете, что здесь все слишком подробно, просто гляньте на картинки, чтобы иметь представление о том, что мы будем делать и переходите к части [**Модели**](/part-2/models.md).

Если же вы новичок, я настоятельно рекомендую продолжить чтение. Это даст вам некоторые навыки моделирования и проектирования Web-приложений. Web-разработка, да и вообще разработка - не только про код.

![Rocket Science](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_Rocket_Science.png)

## Диаграмма использования

Наш проект - нечто наподобие форума. Основная мысль в том, чтобы иметь несколько **"Досок"**, которые будут являться нашими категориями. А в каждой отдельной доске пользователь может начать новое обсуждение, создавая **"топик"**. В этом топике остальные пользователи могут вступить в обсуждение и оставлять ответы.

Нам нужен способ отличить обычного пользователя от администратора, поскольку администраторы могут создавать новые доски. Ниже представлены наши основные способы использования и роли каждого пользователя:

![Use Case Diagram](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/use-case-diagram.png)

*Рис. 1: Диаграмма использования базового функционала*

## Диаграмма классов

Из диаграммы использования мы можем начать размышлять о **"сущностях"** нашего проекта. Сущности - модели, которые мы будем создавать. И это очень близко к данным, которые наше Django-приложение будет обрабатывать.

Для того, чтомы мы могли реализовать варианты использования, приведенные на предыдущем рисунки, нам нужно реализовать следующие модели: **Board**, **Topic**, **Post** и **User**.

![Basic Class Diagram](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/basic-class-diagram.png)

*Рис. 2: Черновик диаграммы классов*

Так же важно подумать, как модели будут отновится друг к другу. Сплошные линии говорят нам, что **Topic** должен содержать поле, определяющее, к какой доске он относится. Также **Post** должен содержать поле для пределение, к какому **Topic** он относится, чтобы мы могли отобразить только те посты, которые относятся к текущему обсужению. Так же нам нужны поля в **Topic** и **Post**, чтобы мы могли узнать, кто создал обсуждение и кто оставил сообщение.

Теперь, когда у нас есть базовое представление необходимых классов, мы можем подумать, какую информацию будет содержать каждая модель. Это может стать сложным очень быстро. Так что постарайтейсб сфокусироваться на главном - данных, которые нам нужны на старте. Остальное будет возможность добавить, используя **миграции**, о которых вы узнаете позже. 

А пока базовым представлением наших данных будут следующие поля:

![Class Diagram](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram.png)

*Рис. 3: Диаграмма классов, описывающая отношения этих классов*

Эта диаграмма классов включает отношения между моделями. Эти линии и стрелки потом будут переведены в поля.

Касаемо модели **Board**, мы начнем с двух полей: **name** и **description**. Поле **name** должно быть уникальным, чтобы избежать дублирования имен досок. **description** - только будет давать подсказку, о чем вообще эта доска.

Модель **Topic** будет состоять из четырех полей: **subject**, **last update** - дата, которая будет определять порядок порядок топиков, **topic starter** - чтобы определить, какой пользователь создал топик, и поле **board**, чтобы определить, к какой доске относится топик.

Модель **Post** будет иметь поле **message**, которое будет использоваться для хранения текста постов ответов, поле **created at** с датой и временем, которое в основном будет использоваться для сортровки сообщений в топике, **updated at** с датой и временем, чтобы информировать пользователей о том, что пост был изменен, если это имело место быть. Так же, у нас должна быть ссылка на **User**: **created by** и **updated by**.

Наконец, модель **User**. В диаграмме классов я только упомянул поля **username**, **password**, **email** и флаг **is superuser**, поскольку это все, что мы будем использовать на данный момент. Так же важно помнить, что мы не должны создавать модель **User**, поскольку Django уже содержит в себе модель в пакете `contrib`. Мы будем использовать ее.

Что касается множественности в диаграмме классов (числа `1`, `0..*` и т.д.), вот как мы будем их читать:

![Class Diagram Board and Topic Association](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-board-topic.png)

**Topic** должен быть связан только с одной(`1`) доской **Board**(что также означает, что оно не может быть `None`), а **Board** может быть связанна с многими **Topic** или ни с одним(`0..*`). Это означает, что **Board** может существовать без единого **Topic**.

![Class Diagram Topic and Post Association](https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-topic-post.png)

A **Topic** should have at least one **Post** (the starter **Post**), and it may also have many **Posts** (`1..*`). A **Post** must be associated with one, and only one **Topic** (`1`).

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