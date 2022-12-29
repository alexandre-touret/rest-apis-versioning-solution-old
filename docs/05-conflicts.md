# Dealing with breaking changes 

## Preamble

Now it is time to move on. 

We just deprecated our [first version](../rest-book) and add new features for our new customers while bringing them wisely to our existing ones!

How to migrate your customers who use the V1 to the V2 ?
Good question!

This first thing to do is to communicate on a regular basis the roadmap and the planned your product End Of Life (EoL) milestones.

By the way, our customer wants having several authors for a same book. 
Currently, one book could only have one author.
This functionality could be considered as a [breaking change](https://en.wiktionary.org/wiki/breaking_change).

Beyond the API definition, this new functionality impacts the whole application. From the OpenAPI description to database schema. 
How could we do that maintaining two versions of our API for our customers?

