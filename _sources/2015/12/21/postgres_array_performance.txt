Postgres Array vs Join benchmark
==================================

.. author:: default
.. categories:: none
.. tags:: python, postgresql, peewee
.. comments::


Here is little experiment to measure postgresql array's performance. For the example problem let us take blog posts and tags. 

Join approach
--------------
This is perhaps more common approach to model posts and tags. So let's define model. Here I am using excellent `Peewee <http://peewee.readthedocs.org>`_. So we have three tables Post, Tag and PostTag. PostTag table maintains all post to tag records.

.. code-block:: python

    class Post(BaseModel):
        title = CharField(default='example title')
    
    
    class Tag(BaseModel):
        name = CharField()
    
    
    class PostTag(BaseModel):
        post = ForeignKeyField(Post)
        tag = ForeignKeyField(Tag)

Array
-----
Postgresql supports array columns. In this model array field Post.tags shall be used to maintain post-tag entries instead of PostTag model. Even `Tag` is not needed in this case.

.. code-block:: python

    class Post(BaseModel):
        title = CharField(default='example title')
        tags = ArrayField(CharField, default=[], index=True)
 
Complete code
-------------

.. code-block:: python

    import random

    from tqdm import tqdm
    from peewee import *
    from myapp import db
    from playhouse.postgres_ext import ArrayField, ForeignKeyField
    
    
    class BaseModel(Model):
        class Meta:
            database = db
            only_save_dirty = True
    
    
    class Post(BaseModel):
        title = CharField(default='example title')
        tags = ArrayField(CharField, default=[], index=True)
    
    
    class Tag(BaseModel):
        name = CharField()
    
    
    class PostTag(BaseModel):
        post = ForeignKeyField(Post)
        tag = ForeignKeyField(Tag)
    
    
    def setup():
        no_of_posts = 25000
        no_of_tags = 10000
        tags_per_post = 15
    
        for t in (PostTag, Tag, Post):
            if t.table_exists():
                t.drop_table()
    
        for t in (Tag, Post, PostTag):
            t.create_table()
    
        tags = [{'name': ('tag-%d' % i)} for i in range(1, no_of_tags)]
        Tag.insert_many(tags).execute()
    
        posts = [{'id': i, 'tags': [('tag-%d' % j) for j in random.sample(range(1, no_of_tags), tags_per_post)]}
                 for i in range(1, no_of_posts)]
        Post.insert_many(posts).execute()
    
        for post in tqdm(posts):
            post_id = post['id']
            post_tags = [{'post': post_id, 'tag': tag.split('-')[1]} for tag in post['tags']]
            PostTag.insert_many(post_tags).execute()
    
        print('Total posts: %d\nTotal tags: %d\nTags per post: %d\n' % (no_of_posts, no_of_tags, tags_per_post))


     def test_join():
        # => SELECT Count(post.id) FROM post INNER JOIN posttag ON (post.id = posttag.post_id) \
        #    WHERE (posttag.tag_id = 8);
        return Post.select().join(PostTag).join(Tag).where(Tag.id == '8').count()
    
    
    def test_array():
        # => SELECT Count("id") FROM post WHERE tags @> '{tag-8}';
        return Post.select().where(Post.tags.contains('tag-8')).count()
    

Needless to say selecting tags for a article would be faster as we are elinimating the joins. But it would be interesting to see that how finding articles for given tag will perform.

And here are the numbers on my machine (Mac Air Ubuntu 15.10 Python 2.7.9).

.. code-block:: bash

    $ python -i bench.py
    >>> setup()
    Total posts: 25000
    Total tags: 1000
    Tags per post: 15

    $ python -mtimeit -s'import bench' 'bench.test_join()'
    100 loops, best of 3: 8.32 msec per loop

    $ python -mtimeit -s'import bench' 'bench.test_array()'
    1000 loops, best of 3: 869 usec per loop
