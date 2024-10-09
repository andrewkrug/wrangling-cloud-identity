# Speak my Language

In this lab we'll discuss ways that developers can interface with the Identity
and Access system by using _plain english_ along with a little bit of code.

## Lab Prerequisites

For this lab you will need a Google Account and use that to sign into 
[Google co-lab](https://colab.research.google.com). Co-lab notebooks are
free to use and great way to test things out without a lot of local setup.

A Linux style terminal with an installation of pmapper:

`pip install git+https://github.com/mosesrenegade/PMapper.git`

> Note if you want to use the graph visualizations in pmapper you'll need
pydot and [graphviz](https://graphviz.org/).  There are a lot of dependencies for the 
dataviz so you may want to skip at the con.

## The quick way to find privescs 

In an authenticated terminal session run the following commands:

1. Gather the data from the target account 
```
pmapper graph create
```

The final output should look like:

```
Graph Data for Account:  654654581435
  # of Nodes:              107 (20 admins)
  # of Edges:              354
  # of Groups:             3
  # of (tracked) Policies: 72
```

2. Now imagine that you want to use that data only to visualize the path to privilege escalations. 
```pmapper visualize --only-privesc --filetype png```

> Note: If you can't run this step examples have been provided in this folder of the output. 

3. Try asking a simple question of pmapper.  Like "who can make an access key"
```pmapper query 'who can do iam:CreateUser'```

4. Now let's imagine that we want to start working on some of those policies but our team doesn't know where to start. One of the most challenging things is actually writing, minimizing, and expanding least privilege policies to understand the impact.

For this exercise we're going to move to google co-lab and use the provided .pynb notebook.  [An example is in this directory](./sample.pynb)





