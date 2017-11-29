Web Board Project

> Оригинал: https://simpleisbetterthancomplex.com/series/2017/09/11/a-complete-beginners-guide-to-django-part-2.html

<p>I don’t know about you, but personally, I learn much more by seeing practical examples and code snippets. For me, it’s
difficult to process a concept where in the examples you read <code class="highlighter-rouge">Class A</code> and <code class="highlighter-rouge">Class B</code>, or when I see the classical
<code class="highlighter-rouge">foo(bar)</code> examples. I don’t want to do that with you.</p>

<p>So, before we get into the fun part, playing with models, views, and everything. Let’s just take a moment and discuss
very briefly about this project we are going to develop.</p>

<p>If you already have experience with Web development and feel it’s too much detail, you can just skim through the
pictures to have an idea what we are going to build and then jump to the <strong>Models</strong> section of this tutorial.</p>

<p>But if you are new to Web development, I highly suggest that you keep reading. It will give you some good insights on
modeling and design of Web applications. Web development, and software development in general, is not just about coding.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_Rocket_Science.png" alt="Rocket Science" /></p>

<h5 id="use-case-diagram">Use Case Diagram</h5>

<p>Our project is a discussion board (a forum). The whole idea is to maintain several <strong>boards</strong>, which will
behave like categories. Then, inside a specific board, a user can start a new discussion by creating a new <strong>topic</strong>.
In this topic, other users can engage in the discussion posting replies.</p>

<p>We will need to find a way to differentiate a regular user from an admin user because only the admins are supposed to
create new boards. Below, an overview of our main use cases and the role of each type of user:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/use-case-diagram.png" alt="Use Case Diagram" /></p>

<p style="text-align: center">
  <small>Figure 1: Use case diagram of the core functionalities offered by the Web Board</small>
</p>

<h5 id="class-diagram">Class Diagram</h5>

<p>From the Use Case Diagram, we can start thinking concerning the <strong>entities</strong> of our project. The entities are the
models we will create, and it’s very closely related to the data our Django app will process.</p>

<p>For us to be able to implement the use cases described in the previous section, we will need to implement at least
the following models: <strong>Board</strong>, <strong>Topic</strong>, <strong>Post</strong>, and <strong>User</strong>.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/basic-class-diagram.png" alt="Basic Class Diagram" /></p>

<p style="text-align: center">
  <small>Figure 2: Draft of the class diagram of the Web Board</small>
</p>

<p>It’s also important to take the time to think about how do models will relate to each other. What the solid lines
are telling us is that, in a <strong>Topic</strong>, we will need to have a field to identify which <strong>Board</strong> it belongs to.
Similarly, the <strong>Post</strong> will need a field to represent which <strong>Topic</strong> it belongs so that we can list in the
discussions only <strong>Posts</strong> created within a specific <strong>Topic</strong>. Finally, we will need fields in both the <strong>Topic</strong> to
know who started the discussion and in the <strong>Post</strong> so we can identify who is posting the reply.</p>

<p>We could also have an association with the <strong>Board</strong> and the <strong>User</strong> model, so we could identify who created a
given <strong>Board</strong>. But this information is not relevant for the application. There are other ways to track this
information, you will see later on.</p>

<p>Now that we have the basic class representation, we have to think what kind of information each of those models
will carry. This sort of thing can get complicated very easily. So try to focus on the important bits. The information
that you need to start the development. Later on, we can improve the model using <strong>migrations</strong>, which you will see in
great detail in the next tutorial.</p>

<p>But for now, this would be a basic representation of our models’ fields:</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram.png" alt="Class Diagram" /></p>

<p style="text-align: center">
  <small>Figure 3: Class diagram emphasizing the relationship between the classes (models)</small>
</p>

<p>This class diagram has the emphasis on the relationship between the models. Those lines and arrows will eventually be
translated into fields later on.</p>

<p>For the <strong>Board</strong> model, we will start with two fields: <strong>name</strong> and <strong>description</strong>. The <strong>name</strong> field has to be
unique, so to avoid duplicated board names. The <strong>description</strong> is just to give a hint of what the board is all about.</p>

<p>The <strong>Topic</strong> model will be composed of four fields: <strong>subject</strong>, <strong>last update</strong> date which will be used to define the
topics ordering, <strong>topic starter</strong> to identify the <strong>User</strong> who started the <strong>Topic</strong>, and a field called <strong>board</strong> to
define which <strong>Board</strong> a specific <strong>Topic</strong> belongs to.</p>

<p>The <strong>Post</strong> model will have a <strong>message</strong> field, which will be used to store the text of the post replies, a <strong>created
at</strong> date and time field mainly used to order the <strong>Posts</strong> within a <strong>Topic</strong>, an <strong>updated at</strong> date and time field
to inform the <strong>Users</strong> when and if a given <strong>Post</strong> has been edited. Like the date and time fields, we will also have
to reference the <strong>User</strong> model: <strong>created by</strong> and <strong>updated by</strong>.</p>

<p>Finally, the <strong>User</strong> model. In the class diagram, I only mentioned the fields <strong>username</strong>, <strong>password</strong>, <strong>email</strong>
and <strong>is superuser</strong> flag because that’s pretty much all we are going to use for now. It’s important to note that we
won’t need to create a <strong>User</strong> model because Django already comes with a built-in <strong>User</strong> model inside the
<strong>contrib</strong> package. We are going to use it.</p>

<p>Regarding the multiplicity in the class diagram (the numbers <code class="highlighter-rouge">1</code>, <code class="highlighter-rouge">0..*</code>, etc.), here’s how you read it:</p>

<p class="u-cf">
<img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-board-topic.png" alt="Class Diagram Board and Topic Association" style="float: right" />

A <strong>Topic</strong> must be associated with exactly one (<code class="highlighter-rouge">1</code>) <strong>Board</strong> (which means it cannot be null), and a <strong>Board</strong> may be
associated with many <strong>Topics</strong> or none (<code class="highlighter-rouge">0..*</code>). Which means a <strong>Board</strong> may exist without a single <strong>Topic</strong>.
</p>

<p class="u-cf">
<img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-topic-post.png" alt="Class Diagram Topic and Post Association" style="float: right" />

A <strong>Topic</strong> should have at least one <strong>Post</strong> (the starter <strong>Post</strong>), and it may also have many <strong>Posts</strong> (<code class="highlighter-rouge">1..*</code>). A
<strong>Post</strong>  must be associated with one, and only one <strong>Topic</strong> (<code class="highlighter-rouge">1</code>).
</p>

<p class="u-cf">
<img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-topic-user.png" alt="Class Diagram Topic and User Association" style="float: right" />

A <strong>Topic</strong> must have one, and only one <strong>User</strong> associated with: the topic starter <strong>User</strong> (<code class="highlighter-rouge">1</code>). And a <strong>User</strong> may
have many or none <strong>Topics</strong> (<code class="highlighter-rouge">0..*</code>).
</p>

<p class="u-cf">
<img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-post-user.png" alt="Class Diagram Post and User Association" style="float: right" />

A <strong>Post</strong> must have one, and only one <strong>User</strong> associated with: <strong>created by</strong> (<code class="highlighter-rouge">1</code>). A <strong>User</strong> may have many or none
<strong>Posts</strong> (<code class="highlighter-rouge">0..*</code>). The second association between <strong>Post</strong> and <strong>User</strong> is a <em>direct association</em> (see the arrow at
the end of the line), meaning we are interested only in one side of the relationship which is what <strong>User</strong> has edited
a given <strong>Post</strong>. It will be translated into the <strong>updated by</strong> field. The multiplicity says <code class="highlighter-rouge">0..1</code>, meaning the
<strong>updated by</strong> field may be null (the <strong>Post</strong> wasn’t edited) and at most may be associated with only one <strong>User</strong>.
</p>

<p>Another way to draw this class diagram is emphasizing the fields rather than in the relationship between the models:</p>

<div id="figure-4">
  <p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/class-diagram-attributes.png" alt="Class Diagram Attributes" /></p>

  <p style="text-align: center">
  <small>Figure 4: Class diagram emphasizing the attributes (fields) of the classes (models)</small>
</p>
</div>

<p>The representation above is equivalent to the previous one, and it’s also closer to what we are going to design using
the Django Models API. In this representation, we can see more clearly that in the <strong>Post</strong> model the associations
<strong>topic</strong>, <strong>created by</strong>, and <strong>updated by</strong> became model fields. Another interesting thing to note is that in the
<strong>Topic</strong> model we have now an <em>operation</em> (a class method) named <strong>posts()</strong>. We are going to achieve this by
implementing a reverse relationship, where Django will automatically execute a query in the database to return a list
of all the <strong>Posts</strong> that belongs to a specific <strong>Topic</strong>.</p>

<p>Alright, enough UML for now! To draw the diagrams presented in this section I used the
<a href="http://staruml.io" target="_blank">StarUML</a> tool.</p>

<h5 id="wireframes">Wireframes</h5>

<p>After spending some time designing the application models, I like to create some wireframes to define what needs to be
done and also to have a clear picture of where we are going.</p>

<p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/Pixton_Comic_Wireframes.png" alt="Wireframes Comic" /></p>

<p>Then based on the wireframes we can gain a deeper understanding of the entities involved in the application.</p>

<p>First thing, we need to show all the boards in the homepage:</p>

<div id="figure-5">
  <p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/wireframe-boards.png" alt="Wireframe Boards" /></p>
  <p style="text-align: center">
    <small>Figure 5: Boards project wireframe homepage listing all the available boards.</small>
  </p>
</div>

<p>If the user clicks on a link, say in the Django board, it should list all the topics:</p>

<div id="figure-6">
  <p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/wireframe-topics.png" alt="Wireframe Topics" /></p>
  <p style="text-align: center">
    <small>Figure 6: Boards project wireframe listing all topics in the Django board.</small>
  </p>
</div>

<p>Here we have two main paths: either the user clicks on the “new topic” button to create a new topic, or the user clicks
on a topic to see or engage in a discussion.</p>

<p>The “new topic” screen:</p>

<div id="figure-7">
  <p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/wireframe-new-topic.png" alt="Wireframe New Topic" /></p>
  <p style="text-align: center">
    <small>Figure 7: New topic screen</small>
  </p>
</div>

<p>Now the topic screen, displaying the posts and discussions:</p>

<div id="figure-8">
  <p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/wireframe-posts.png" alt="Wireframe Posts" /></p>
  <p style="text-align: center">
    <small>Figure 8: Topic posts listing screen</small>
  </p>
</div>

<p>If the user clicks on the reply button, they will see the screen below, with a summary of the posts in reverse order
(newest first):</p>

<div id="figure-9">
  <p><img src="https://simpleisbetterthancomplex.com/media/series/beginners-guide/1.11/part-2/wireframe-reply.png" alt="Wireframe Reply" /></p>
  <p style="text-align: center">
    <small>Figure 9: Reply topic screen</small>
  </p>
</div>

<p>To draw your wireframes you can use the <a href="https://draw.io" target="_blank">draw.io</a> service, it’s free.</p>

<hr />
