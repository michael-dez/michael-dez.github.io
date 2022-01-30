---
title: "Agent to Agent Calls in Connect: Lambda"
author_profile: true
excerpt: "/"It's JSON all the way down./" -Marco Polo"
header:
  overlay_image: /assets/img/home-header.jpg
  actions:
    - label: "View on Github"
      url: "https://github.com/michael-dez/awsconnect-extensions"
---
## Overview
Before I go into the Lambda I want to take a minute to explain some changes I'm making to future posts. Firstly, I've found it very difficult to maintain a consistent pace of writing. The last thing I want to do after finishing a project is a write-up of the project because I'm already thinking about the next thing I want to learn. I still want to document what I'm learning in my free time and I think I would especially benefit from better note taking. So I intend to post more frequently, with a greater focus on brief and instructive writing. My intention is to take better notes as I work so that the posts can be released as the work is completed. This will help tremendously with the writing process as writing about things I did a month prior really slows me down, and makes the thought of sitting down to write somewhat dreadful. I also think pragmatic, instructive posts will be the most useful to both myself and readers... after this long winded write up. With that, to the Lambda.
## The First Steps
It's a little overwhelming deciding how much work to put into a project like this. I went with CloudShell + Vim because it's free and it provides me a text editor I'm familiar with and a terminal that's easy to access. I like CloudShell, but a problem I have is that often the terminal buffer will get mixed in with Vim buffers or vice versa. Another is all of the vim, tmux, and terminal keyboard shortcuts that can't be used because they are bound to browser keyboard shortcuts. I was very disappointed trying to use Ctrl+W to change Vim windows. 
The first function I wrote was to populate the DynamoDB Table with 10,000 available extensions. Took a bit over five minutes to run and then I learned how easy and awesome batch writes are to use which really lowered the running time.
```python
def initialize_table():
'''Initializes table with all available extensions.'''
    with table.batch_writer() as writer:
        for x in range(10000):
            extension = str(x)
            extension = extension.zfill(4)

            newItem = {
            "pk": extension,
            "sk": "nu",
            "sk_value": extension
            }

            response = writer.put_item(Item=newItem)
    return
```
## Getting All the Data
The first piece of information I wanted to have available was a current list of agents from Connect. It should be noted that there is a service quota for API calls to Connect so adding exponential back off would be a good idea.
```python
def get_users():
    '''Gets list of connect users, returns as set of usernames.'''
    users = set() 
    response = connect.list_users(
    InstanceId=CONNECT_IID,
    MaxResults=10
    )

    while "NextToken" in response:
        addUsers = response.get('UserSummaryList')

    for _ in addUsers:
        users.add(_.get('Username'))

    response = connect.list_users(
        NextToken=response["NextToken"],
        InstanceId=CONNECT_IID,
        MaxResults=10)

    for _ in response.get('UserSummaryList'):
        users.add(_.get('Username'))

    return users
```
The results of the `list_users()` API call depends on the MaxResults parameter and the number of users to list. Getting a complete list requires either repeated calls utilizing the returned `NextToken` or using a `Paginator` which returns an iterator of the command. I chose to store `users` as a set because it will be referenced to determine whether a database entry is current or not. For this it should perform quickly.
Something I found interesting and I didn't recall being covered in my AWS Developer course is that some AWS Service APIs offer both a `Client` api that represents the service at a low level and `Resource` api that allows the use of common resources, such as DynamoDB Tables, as objects. I found the `Resource` api simpler to use.
## Putting It All Together
I created additional functions for setting an extension, checking if an agent already had an extension, and to export a list of extensions to S3 after updating the table. The last addition was the required `lambda_handler` which directly calls the main `update_db()` function.
```python
def update_db():
    '''Helper function to make lambda_handler as small as possible.'''
    users = get_users() 
    get_unused_ext()
    if not unused and users:
        initialize_table()

    for _ in users:
        set_extension(_)

    get_db_users()

    if len(db_users) > len(users):
        for _ in db_users:
            if _ not in users:
            remove_user(db_users[_])
            logger.info(_ + "[" + db_users[_] + "]: removed")
            db_users.pop(_)

    if BUCKET_NAME: 
    export_s3()

    return
```
## Conclusion
I tried to keep functions small including the main `update_db()` function at the recommendation of a friend and I found everything to be a lot more readable. He also recommended I read the book Clean Code by Robert Martin and I'm really looking forward to it. I did not end up using any classes in the Lambda. I thought about creating an `Agent` class but the relevant 'agents' would be from Connect and I didn't see any immediate value in the extra work. I did get some practice when making a script to generate a csv of test agent data to upload through the Connect console and it felt amazing being able to quickly produce code that got a potentially long and tedious job done quicker than attempting to do it by hand. Overall, I loved the experience. Debugging was much easier than it was in my experience using Java or Visual Basic. The documentation was excellent and approachable. Working on the project was (almost) always fun and as I started to improve, it felt empowering to be able to work quickly and at a high level.
