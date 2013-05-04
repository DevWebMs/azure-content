Application .NET Multi-Tiers utilisant les Tables de stockage, Queues et Blobs - 1 sur 5
========================================================================================

Cette série de tutoriels démontre comment créer une application web multi-tiers
ASP.NET MVC qui utilise les Tables de Stockage de Windows Azure, les Queues et
les Blobs, et comment déployer l'application sur le service cloud de Windows
Azure. Le tutoriel n'assume aucune expérience précédente avec Windows Azure. En
complétant cette série, vous apprendrez comment construire une application web
guidée par les donées robuste, et évolutive (scalable) qui se déploie sur le
cloud.

Dans cette série de tutoriel, vous apprendrez :

-   Comment préparer votre machine au dévelopement pour Windows Azure en
    installant le SDK Windows Azure.

-   Comment créer un projet cloud dans visual studio avec un web role MVC 4 et
    deux worker roles.

-   Comment publier le projet sur un service cloud Windows Azure.

-   Comment publier le projet MVC sur un Windows Azure Web Site si vous le
    préférez, tout en utilisant les worker roles sur le Cloud Service.

-   Comment utiliser le service Windows Azure Queue pour la communication entre
    les tiers et les worker roles.

-   Comment utiliser le service Windows Azure Table storage en tant que stockage
    de données hautement évolutive pour des données structurées
    non-relationelles.

-   Comment utiliser le service Windows Azure Blob storage pour stocker des
    fichiers sur le cloud.

-   Comment voir et éditer les tables, queues et blobs de Windows Azure en
    utilisant Visual Studio ou Azure Storage Explorer.

-   Comment utiliser SendGrid pour envoyer des emails.

-   Comment configurer le traçage et consulter les données de traçage.

-   Comment évoluer une application en augmentant le nombre d'instance des
    worker roles.

L'application que vous aller construire est un service de liste d'email. le
front-end de l'application multi-tiers inclue des pages web que les
administrateurs du service peuvent utiliser pour gérer les listes d'emails.

![](<../Media/mtas-mailing-list-index-page.png>)

![](<../Media/mtas-subscribers-index-page.png>)

Ainsi que des page pour les administrateurs utilisent pour créer les messages a
envoyé à une liste d'emails

![](<../Media/mtas-message-index-page.png>)

![](<../Media/mtas-message-create-page.png>)

Les clients du services sont des entreprises qui offrent l'opportunité à leurs
utilisateurs de s'inscrire à une liste de diffusion sur leur site internet. Par
exemple, un administrateur met en place une liste de diffusion pour les annonces
du Département de Sociologie de l'Université Contoso. Quand un étudiant
intéressé par les annonces de ce département clique un lien du site internet de
l'Université Contoso, le site de l'Université appelle un service web de
l'application de Service Email sur Windows Azure. Le service déclenche l'envoie
d'un email à l'étudiant. Cet email contient un lien hypertexte, et lorsque le
destinataire clique sur le lien, une page l'accueillant au Service d'Annonce du
Département de Sociologie est affichée.

![](<../Media/mtas-subscribe-email.png>)

![](<../Media/mtas-subscribe-confirmation-page.png>)

Chaque email envoyé par le service (sauf les confirmations d'abonnement) inclue
un lien hypertexte qui permet de se désabonner. Si un déstinataire clique ce
lien, une page web affiché demandant la confirmation de désabonnement.

![](<../Media/mtas-unsubscribe-query-page.png>)

Si le destinataire clique sur le bouton **Confirmer**, une page confirmant que
la person a été retirée de la liste de diffusion est affichée.

![](<../Media/mtas-unsubscribe-confirmation-page.png>)

Voici une liste des tutoriels accompagné du résumé de leur contenu :

1.  **Introduction à l'application Service Email Windows Azure** (ce tutoriel).
    Une vue d'ensemble de l'application et de son architecture.

2.  [Configurer et Déployer l'application Service Email Windows Azure][34].
    Comment télécharger l'application exemple, la configurer, la tester
    localement, la déployer, et la tester sur le cloud.

[34]: </en-us/develop/net/tutorials/multi-tier-web-site/2-download-and-run/>

1.  [Construire le web role pour l'application Service Email Windows Azure][2].
    Comment construire les composants MVC 4 de l'application et les tester
    localement.

[2]: </en-us/develop/net/tutorials/multi-tier-web-site/3-web-role/>

1.  [Construire le worker role A (scheduler d'email) pour l'application Service
    Email Windows Azure][3]. Comment construire la partie serveur qui ajoute des
    work items dans la queue pour l'envoie d'emails, et la tester localement.

[3]: </en-us/develop/net/tutorials/multi-tier-web-site/4-worker-role-a/>

1.  [Construire le worker role B (l'expéditeur d'email) pour l'application
    Service Email Windows Azure][4]. Comment construire la partie serveur qui
    consume les work items de la queue pour l'envoie d'emails, et la tester
    localement.

[4]: </en-us/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/>

Si vous voulez seulement télécharger l'application et l'essayer, tout ce dont
vous avez besoin sont les 2 premiers tutoriels. Si vous voulez voir toutes les
étapes de la construction d'une telle application en partant de zéro, continuez
avec les trois derniers tutoriels après avoir suivi les deux premiers.

Nous avons choisi un service de liste de diffusion d'email pour cet exemple
d'application parce-que c'est le type d'application qui a besoin d'être robuste
et évolutive (scalable), deux propriétés qui la rende particulièrement adéquate
pour Windows Azure.

### Robuste

Si un serveur échoue pendant qu'il envoie des emails à une large liste, vous
voulez avoir la possibilité de démarrer un nouveau serveur facilement et
rapidement, et vous voulez que l'application redémarre la où elle s'était
arrêtée sans perdre ni dupliquer d'emails. Les instances de worker roles ou de
web roles sur Windows Azure Cloud Service sont automatiquement remplacées si
elles échouent. Et les Queues et Tables de Windows Azure Storage fournissent un
moyen de communication serveur-à-serveurs qui peut survivre aux échecs sans
perdre le travail en cours.

### Scalable

An email service also must be able to handle spikes in workload, since sometimes
you are sending emails to small lists and sometimes to very large lists. In many
hosting environments, you have to purchase and maintain sufficient hardware to
handle the spikes in workload, and you’re paying for all that capacity 100% of
the time although you might only use it 5% of the time. With Windows Azure, you
pay only for the amount of computing power that you actually need for only as
long as you need it. To scale up for a large mailing, you just change a
configuration setting to increase the number of servers you have available to
process the workload, and this can be done programmatically. For example, you
could configure the application so that if the number of work items waiting in
the queue exceeds a certain number, Windows Azure automatically spins up
additional instances of the worker role that processes those work items.

The front-end stores email lists and messages to be sent to them in Windows
Azure tables. When an administrator schedules a message to be sent, a table row
containing the scheduled date and other data such as the subject line is added
to the `message` table. A worker role periodically scans the `message` table
looking for messages that need to be sent (we’ll call this worker role A).

When worker role A finds a message needing to be sent, it does the following
tasks:

-   Gets all the email addresses in the destination email list.

-   Puts the information needed to send each email in the `message` table.

-   Creates a queue work item for each email that needs to be sent.

A second worker role (worker role B) polls the queue for work items. When worker
role B finds a work item, it processes the item by sending the email, and then
it deletes the work item from the queue. The following diagram shows these
relationships.

![](<../Media/mtas-worker-roles-a-and-b.png>)

No emails are missed if worker role B goes down and has to be restarted, because
a queue work item for an email isn’t deleted until after the email has been
sent. The back-end also implements table processing that prevents multiple
emails from getting sent in case worker role A goes down and has to be
restarted. In that case, multiple queue work items might be generated for a
given destination email address. But for each destination email address, a row
in the `message` table tracks whether the email has been sent. Depending on the
timing of the restart and email processing, worker A uses this row to avoid
creating a second queue work item, or worker B uses this row to avoid sending a
second email.

![](<../Media/mtas-message-processing.png>)

Worker role B also polls a subscription queue for work items put there by the
Web API service method for new subscriptions. When it finds one, it sends the
confirmation email.

![](<../Media/mtas-subscribe-diagram.png>)

The Windows Azure Email Service application stores data in Windows Azure Storage
tables. Windows Azure tables are a NoSQL data store, not a relational database
like [Windows Azure SQL Database][5]. That makes them a good choice when
efficiency and scalability are more important than data normalization and
relational integrity. For example, in this application, one worker role creates
a row every time a queue work item is created, and another one retrieves and
updates a row every time an email is sent, which might become a performance
bottleneck if a relational database were used. Additionally, Windows Azure
tables are cheaper than Windows Azure SQL. For more information about Windows
Azure tables, see the resources that are listed at the end of [the last tutorial
in this series][6].

[5]: <http://msdn.microsoft.com/en-us/library/windowsazure/ee336279.aspx>

[6]: </en-us/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/>

The following sections describe the contents of the Windows Azure tables that
are used by the Windows Azure Email Service application. For a diagram that
shows the tables and their relationships, see the [Windows Azure Email Service
data diagram][7] later in this page.

[7]: <#datadiagram>

### mailinglist table

The `mailinglist` table stores information about mailing lists and the
subscribers to mailing lists. (The Windows Azure table naming convention best
practice is to use all lower-case letters.) Administrators use web pages to
create and edit mailing lists, and clients and subscribers use a set of web
pages and a service method to subscribe and unsubscribe.

In NoSQL tables, different rows can have different schemas, and this flexibility
is commonly used to make one table store data that would require multiple tables
in a relational database. For example, to store mailing list data in SQL
Database you could use three tables: a `mailinglist` table that stores
information about the list, a `subscriber` table that stores information about
subscribers, and a `mailinglistsubscriber` table that associates mailing lists
with subscribers and vice versa. In the NoSQL table in this application, all of
those functions are rolled into one table named `mailinglist`.

In a Windows Azure table, every row has a *partition key* and a *row key* that
uniquely identifies the row. The partition key divides the table up logically
into partitions. Within a partition, the row key uniquely identifies a row.
There are no secondary indexes; therefore to make sure that the application will
be scalable, it is important to design your tables so that you can always
specify partition key and row key values in the Where clause of queries.

The partition key for the `mailinglist` table is the name of the mailing list.

The row key for the `mailinglist` table can be one of two things: the constant
“mailinglist” or the email address of the subscriber. Rows that have row key
“mailinglist” include information about the mailing list. Rows that have the
email address as the row key have information about the subscribers to the list.

In other words, rows with row key “mailinglist” are equivalent to a
`mailinglist` table in a relational database. Rows with row key = email address
are equivalent to a `subscriber` table and a `mailinglistsubscriber` association
table in a relational database.

Making one table serve multiple purposes in this way facilitates better
performance. In a relational database three tables would have to be read, and
three sets of rows would have to be sorted and matched up against each other,
which takes time. Here just one table is read and its rows are automatically
returned in partition key and row key order.

The following grid shows row properties for the rows that contain mailing list
information (row key = “MailingList”).

The following grid shows row properties for the rows that contain subscriber
information for the list (row key = email address).

The following list shows an example of what data in the table might look like.

### message table

The `message` table stores information about messages that are scheduled to be
sent to a mailing list. Administrators create and edit rows in this table using
web pages, and the worker roles use it to pass information about each email from
worker role A to worker role B.

The partition key for the message table is the date the email is scheduled to be
sent, in yyyy-mm-dd format. This optimizes the table for the query that is
executed most often against this table, which selects rows that have
`ScheduledDate` of today or earlier. However, it does creates a potential
performance bottleneck, because Windows Azure Storage tables have a maximum
throughput of 500 entities per second for a partition. For each email to be
sent, the application writes a `message` table row, reads a row, and deletes a
row. Therefore the shortest possible time for processing 1,000,000 emails
scheduled for a single day is almost two hours, regardless of how many worker
roles are added in order to handle increased loads.

The row key for the `message` table can be one of two things: the constant
“message” plus a unique key for the message called the `MessageRef`, or the
`MessageRef` value plus the email address of the subscriber. Rows that have row
key that begins with “message” include information about the message, such as
the mailing list to send it to and when it should be sent. Rows that have the
`MessageRef` and email address as the row key have all of the information needed
to send an email to that email address.

In relational database terms, rows with row key that begins with “message” are
equivalent to a `message` table. Rows with row key = `MessageRef` plus email
address are equivalent to a join query view that contains `mailinglist`,
`message`, and `subscriber` information.

The following grid shows row properties for the `message` table rows that have
information about the message itself.

When worker role A creates a queue message for an email to be sent to a list, it
creates an email row in the `message` table. When worker role B sends the email,
it moves the email row to the `messagearchive` table and updates the `EmailSent`
property to `true`. When all of the email rows for a message in Processing
status have been archived, worker role A sets the status to Completed and moves
the `message` row to the `messagearchive` table.

The following grid shows row properties for the email rows in the `message`
table.

There is redundant data in these rows, which you would typically avoid in a
relational database. But in this case you are trading some of the disadvantages
of redundant data for the benefit of greater processing efficiency and
scalability. Because all of the data needed for an email is present in one of
these rows, worker role B only needs to read one row in order to send an email
when it pulls a work item off the queue.

You might wonder where the body of the email comes from. These rows don’t have
blob references for the files that contain the body of the email, because that
value is derived from the `MessageRef` value. For example, if the `MessageRef`
is 634852858215726983, the blobs are named 634852858215726983.htm and
634852858215726983.txt.

The following list shows an example of what data in the table might look like.

\<br/\> \<br/\>

### messagearchive table

One strategy for making sure that queries execute efficiently, especially if you
have to search on fields other than `PartitionKey` and `RowKey`, is to limit the
size of the table. The query in worker role A that checks to see if all emails
have been sent for a message needs to find email rows in the `message` table
that have `EmailSent` = false. The `EmailSent` value is not in the PartitionKey
or RowKey, so this would not be an efficient query for a message with a large
number of email rows. Therefore, the application moves email rows to the
`messagearchive` table as the emails are sent. As a result, the query to check
if all emails for a message have been sent only has to query the message table
on `PartitionKey` and `RowKey` because if it finds any email rows for a message
at all, that means there are unsent messages and the message can’t be marked
`Complete`.

The schema of rows in the `messagearchive` table is identical to that of the
`message` table. Depending on what you want to do with this archival data, you
could limit its size and expense by reducing the number of properties stored for
each row, and by deleting rows older than a certain age.

Windows Azure queues facilitate communication between tiers of this multi-tier
application, and between worker roles in the back-end tier. Queues are used to
communicate between worker role A and worker role B in order to make the
application scalable. Worker role A could create a row in the Message table for
each email, and worker role B could scan the table for rows representing emails
that haven’t been sent, but you wouldn’t be able to add additional instances of
worker role B in order to divide up the work. The problem with using table rows
to coordinate the work between worker role A and worker role B is that you have
no way of ensuring that only one worker role instance will pick up any given
table row for processing. Queues give you that assurance. When a worker role
instance pulls a work item off a queue, the queue service makes sure that no
other worker role instance can pull the same work item. This exclusive lease
feature of Windows Azure queues facilitates sharing a workload among multiple
instances of a worker role.

Windows Azure also provides the Service Bus queue service. For more information
about Windows Azure Storage queues and Service Bus queues, see the resources
that are listed at the end of [the last tutorial in this series][8].

[8]: </en-us/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/>

The Windows Azure Email Service application uses two queues, named
`AzureMailQueue` and `AzureMailSubscribeQueue`.

### AzureMailQueue

The `AzureMailQueue` queue coordinates the sending of emails to email lists.
Worker role A places a work item on the queue for each email to be sent, and
worker role B pulls a work item from the queue and sends the email.

A queue work item contains a comma-delimited string that consists of the
scheduled date of the message (partition key to the `message` table) and the
`MessageRef` and `EmailAddress` values (row key to the `message` table) values,
plus a flag indicating whether the item is created after the worker role went
down and restarted, for example:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  2012-10-15,634852858215726983,student1@contoso.edu,0
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Worker role B uses these values to look up the row in the `message` table that
contains all of the information needed to send the email. If the restart flag
indicates a restart, worker B makes sure the email has not already been sent
before sending it.

When traffic spikes, the Cloud Service can be reconfigured so that multiple
instances of worker role B are instantiated, and each of them can independently
pull work items off the queue.

### AzureMailSubscribeQueue

The `AzureMailSubscribeQueue` queue coordinates the sending of subscription
confirmation emails. In response to a service method call, the service method
places a work item on the queue. Worker role B pulls the work item from the
queue and sends the subscription confirmation email.

A queue work item contains the subscriber GUID. This value uniquely identifies
an email address and the list to subscribe it to, which is all that worker role
B needs to send a confirmation email. As explained earlier, this requires a
query on a field that is not in the `PartitionKey` or `RowKey`, which is
inefficient. To make the application more scalable, the `mailinglist` table
would have to be restructured to include the subscriber GUID in the `RowKey`.

The following diagram shows the tables and queues and their relationships.

![](<../Media/mtas-datadiagram.png>)

Blobs are “binary large objects.” The Windows Azure Blob service provides a
means for uploading and storing files in the cloud. For more information about
Windows Azure blobs, see the resources that are listed at the end of [the last
tutorial in this series][9].

[9]: </en-us/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/>

Windows Azure Mail Service administrators put the body of an email in HTML form
in an *.htm* file and in plain text in a *.txt* file. When they schedule an
email, they upload these files in the **Create Message** web page, and the
ASP.NET MVC controller for the page stores the uploaded file in a Windows Azure
blob.

Blobs are stored in blob containers, much like files are stored in folders. The
Windows Azure Mail Service application uses a single blob container, named
**azuremailblobcontainer**. The name of the blobs in the container is derived by
concatenating the MessageRef value with the file extension, for example:
634852858215726983.htm and 634852858215726983.txt.

Since both HTML and plain text messages are essentially strings, we could have
designed the application to store the email message body in string properties in
the `Message` table instead of in blobs. However, there is a 64K limit on the
size of a property in a table row, so using a blob avoids that limitation on
email body size. (64K is the maximum total size of the property; after allowing
for encoding overhead, the maximum string size you can store in a property is
actually closer to 48k.)

When you download the Windows Azure Email Service, it is configured so that the
front-end and back-end all run in a single Windows Azure Cloud Service.

![](<../Media/mtas-architecture-overview.png>)

An alternative architecture is to run the front-end in a Windows Azure Web Site.

![](<../Media/mtas-alternative-architecture.png>)

Keeping all components in a cloud service simplifies configuration and
deployment. If you create the application with the ASP.NET MVC front end in a
Windows Azure Web Site, you will have two deployments, one to the Windows Azure
Web Site and one to the Windows Azure Cloud Service. In addition, Windows Azure
Cloud Service web roles provide the following features that are unavailable in
Windows Azure Web Sites:

-   Support for custom and wildcard certificates.

-   Full control over how IIS is configured. Many IIS features cannot be enabled
    on Windows Azure Web sites. With Windows Azure web roles, you can define a
    startup command that runs the [AppCmd][10] program to modify IIS settings
    that cannot be configured in your *Web.config* file. For more information,
    see [How to Configure IIS Components in Windows Azure][11] and [How to Block
    Specific IP Addresses from Accessing a Web Role ][12].

[10]: <http://www.iis.net/learn/get-started/getting-started-with-iis/getting-started-with-appcmdexe>

[11]: <http://msdn.microsoft.com/en-us/library/windowsazure/gg433059.aspx>

[12]: <http://msdn.microsoft.com/en-us/library/windowsazure/jj154098.aspx>

-   Support for automatically scaling your web application by using the
    [Autoscaling Application Block][13].

[13]: </en-us/develop/net/how-to-guides/autoscaling/>

-   The ability to run elevated startup scripts to install applications, modify
    registry settings, install performance counters, etc.

-   Network isolation for use with [Windows Azure Connect][14] and [Windows
    Azure Virtual Network][15].

[14]: <http://msdn.microsoft.com/en-us/library/windowsazure/gg433122.aspx>

[15]: <http://msdn.microsoft.com/en-us/library/windowsazure/jj156007.aspx>

-   Remote desktop access for debugging and advanced diagnostics.

-   Rolling upgrades with [Virtual IP Swap][16]. This feature swaps the content
    of your staging and production deployments.

[16]: <http://msdn.microsoft.com/en-us/library/windowsazure/ee517253.aspx>

The alternative architecture might offer some cost benefits, because a Windows
Azure Web Site might be less expensive for similar capacity compared to a web
role running in a Cloud Service. Later tutorials in the series explain
implementation details that differ between the two architectures.

For more information about how to choose between Windows Azure Web Sites and
Windows Azure Cloud Services, see [Windows Azure Execution Models][17].

[17]: <http://www.windowsazure.com/en-us/manage/windows/fundamentals/compute/>

This section provides a brief overview of costs for running the sample
application in Windows Azure, given rates in effect when the tutorial was
published in December of 2012. Before making any business decisions based on
costs, be sure to check current rates on the following web pages:

-   [Windows Azure Pricing Calculator][18]

[18]: <http://www.windowsazure.com/en-us/pricing/calculator/>

-   [SendGrid Windows Azure][19]

[19]: <http://www.windowsazure.com/en-us/store/service/?id=f131eadb-7aa3-401a-a2fb-1c7e71f45c3c>

Costs are affected by the number of web and worker role instances you decide to
maintain. In order to qualify for the [Azure Cloud Service 99.95% Service Level
Agreement (SLA)][20], you must deploy two or more instances of each role. One of
the reasons you must run at least two role instances is because the virtual
machines that run your application are restarted approximately twice per month
for operating system upgrades. (For more information on OS Updates, see [Role
Instance Restarts Due to OS Upgrades][21].)

[20]: <https://www.windowsazure.com/en-us/support/legal/sla/>

[21]: <http://blogs.msdn.com/b/kwill/archive/2012/09/19/role-instance-restarts-due-to-os-upgrades.aspx>

The work performed by the two worker roles in this sample is not time critical
and so does not need the 99.5% SLA. Therefore, running a single instance of each
worker role is feasible so long as one instance can keep up with the work load.
The web role instance is time sensitive, that is, users expect the web site to
not have any down time, so a production application should have at least two
instances of the web role.

The following table shows the costs for the default architecture for the Windows
Azure Email Service sample application assuming a minimal workload. The costs
shown are based on using an extra small (shared) virtual machine size. The
default virtual machine size when you create a Visual Studio cloud project is
small, which is about six times more expensive than the extra small size.

As you can see, role instances are a major component of the overall cost. Role
instances incur a cost even if they are stopped; you must delete a role instance
to not incur any charges. One cost saving approach would be to move all the code
from worker role A and worker role B into one worker role. For these tutorials
we deliberately chose to implement two worker instances in order to simplify
scale out. The work that worker role B does is coordinated by the Windows Azure
Queue service, which means that you can scale out worker role B simply by
increasing the number of role instances. (Worker role B is the limiting factor
for high load conditions.) The work performed by worker role A is not
coordinated by queues, therefore you cannot run multiple instances of worker
role A. If the two worker roles were combined and you wanted to enable scale
out, you would need to implement a mechanism for ensuring that worker role A
tasks run in only one instance. (One such mechanism is provided by
[CloudFx][22]. See the [WorkerRole.cs sample][23].)

[22]: <http://nuget.org/packages/Microsoft.Experience.CloudFx>

[23]: <http://code.msdn.microsoft.com/windowsazure/CloudFx-Samples-60c3a852/sourcecode?fileId=57087&pathId=528472169>

It is also possible to move all of the code from the two worker roles into the
web role so that everything runs in the web role. However, performing background
tasks in ASP.NET is not supported or considered robust, and this architecture
would complicate scalability. For more information see [The Dangers of
Implementing Recurring Background Tasks In ASP.NET][24]. See also [How to
Combine a Worker and Web Role in Windows Azure][25] and [Combining Multiple
Azure Worker Roles into an Azure Web Role][26].

[24]: <http://haacked.com/archive/2011/10/16/the-dangers-of-implementing-recurring-background-tasks-in-asp-net.aspx>

[25]: <http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2010/12/how-to-combine-worker-and-web-role-in.html>

[26]: <http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html>

Another architecture alternative that would reduce cost is to use the
[Autoscaling Application Block][27] to automatically deploy worker roles only
during scheduled periods, and delete them when work is completed. For more
information on autoscaling, see the links at the end of [the last tutorial in
this series][28].

[27]: </en-us/develop/net/how-to-guides/autoscaling/>

[28]: </en-us/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/>

Windows Azure in the future might provide a notification mechanism for scheduled
reboots, which would allow you to only spin up an extra web role instance for
the reboot time window. You wouldn’t qualify for the 99.95 SLA, but you could
reduce your costs by almost half and ensure your web application remains
available during the reboot interval.

In a production application you would implement an authentication and
authorization mechanism like the ASP.NET membership system for the ASP.NET MVC
web front-end, including the ASP.NET Web API service method. There are also
other options, such as using a shared secret, for securing the Web API service
method. Authentication and authorization functionality has been omitted from the
sample application to keep it simple to set up and deploy. (The second tutorial
in the series shows how to implement IP restrictions so that unauthorized
persons can’t use the application when you deploy it to the cloud.)

For more information about how to implement authentication and authorization in
an ASP.NET MVC web project, see the following resources:

-   [Authentication and Authorization in ASP.NET Web API][29]

[29]: <http://www.asp.net/web-api/overview/security/authentication-and-authorization/authentication-and-authorization-in-aspnet-web-api>

-   [Using Forms Authentication][30]

[30]: <http://msdn.microsoft.com/en-us/library/ff398049(VS.98).aspx>

-   [Music Store Part 7: Membership and Authorization][31]

[31]: <http://www.asp.net/mvc/tutorials/mvc-music-store/mvc-music-store-part-7>

**Note**: We planned to include a mechanism for securing the Web API service
method by using a shared secret, but that was not completed in time for the
initial release. Therefore the third tutorial does not show how to build the Web
API controller for the subscription process. We hope to include instructions for
implementing a secure subscription process in the next version of this tutorial.
Until then, you can test the application by using the administrator web pages to
subscribe email addresses to lists.

In the [next tutorial][32], you’ll download the sample project, configure your
development environment, configure the project for your environment, and test
the project locally and in the cloud. In the following tutorials you’ll see how
to build the project from scratch.

[32]: </en-us/develop/net/tutorials/multi-tier-web-site/2-download-and-run/>

For links to additional resources for working with Windows Azure Storage tables,
queues, and blobs, see the end of [the last tutorial in this series][33].

[33]: </en-us/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/>
