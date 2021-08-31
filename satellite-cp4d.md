### **Kevin Collins**

Senior Technical Specialist

IBM Public Cloud - Americas

kevincollins\@us.ibm.com

**Kunal Malhotra**
------------------

#### Cloud Platform Engineer

IBM Cloud MEA

kunal.malhotra3\@ibm.com

Let's say you have deployed a RedHat OpenShift Cluster via IBM Cloud
Satellite in a private isolated network in your datacenter and now you
have deployed certain applications on this OpenShift Cluster which are
required to be exposed to the public Internet. According to the current
design the only way it is possible to expose the application to the
public Internet is either you expose your whole satellite location to
the Internet or use networking infrastructure such (load balancer,
http/https/TLS proxy etc.).

Exposing the whole satellite location brings up a huge security concern
i.e. anybody from the outside world can access you satellite control
plane in the datacenter or the OpenShift worker nodes themselves and
this is not ideal for any enterprise. Hence using networking
infrastructure make much more sense for both security and operational
points to expose applications to the Internet. This involves having a
networking device outside of the cluster and configuring the load
balancer to access a NodePort service on your OpenShift Cluster. Not
only will you find this method more secure, it also scales better. In
tutorial we will show you how to NAT the traffic coming from the
Internet to your internal private network via a load balancer so that
your application is accessible via the internet.

Note: This step-by-step tutorial have been performed on IBM Cloud, but
steps should be similar on other public cloud providers and on-prem.

Create a load balancer service with your cloud provider.
========================================================

**IBM Cloud Example:**

From the IBM Cloud menu, navigate to **VPC Infrastructure** and under
**Network**, select **Load Balancers**, and click **Create**.

![Graphical user interface, application Description automatically
generated](.//media/image1.png){width="3.6857141294838147in"
height="1.5530391513560804in"}

Configure the Load Balancer
---------------------------

Give the load balancer a **name**, select **Application Load Balancer**,
select the **Region** where your satellite hosts are, select your
**virtual private cloud** your hosts are in, and finally select **all
subnets** for your VPC.

![Graphical user interface, application, Teams Description automatically
generated](.//media/image2.png){width="4.919047462817148in"
height="2.7496216097987753in"}

Create a backend pool
---------------------

In OpenShift, get the nodeport of the service you want to expose to the
Internet.

**Example to Test:**

Create a new openshift application

oc new-app openshift/hello-openshift

This creates a pod and a cluster ip service.

Edit the service, search for ClusterIP and change it to NodePort. Save
the service.

oc edit svc hello-openshift

Note the port of the service.

![](.//media/image3.png){width="6.5in" height="0.46041666666666664in"}

In this case, we want to create a load balancer for the service running
on port 8080 which maps to port 30268 in the cluster.

Create a backend pool
---------------------

Back on the create load balancer page, click on **New Pool** in the
backend pool section.

![Graphical user interface Description automatically generated with low
confidence](.//media/image4.png){width="5.604761592300962in"
height="1.0263418635170605in"}

Following the above example, we will use the name hello-openshift and
for Health Port, but the port number we got in the previous step ... in
this case 30268.

![Graphical user interface, application, Teams Description automatically
generated](.//media/image5.png){width="2.6714282589676293in"
height="5.823670166229221in"}

**Attach Virtual Hosts to the Backend Pool.**

Under the backend pool section, click on **attach**.

![A picture containing graphical user interface Description
automatically generated](.//media/image6.png){width="6.5in"
height="0.8673611111111111in"}

On the next screen select the first subnet, and then select the first
worker node instance, and use the same port number ( 30268 in our case
). Click add and **repeat the same steps for every subnet and every
worker node** in the cluster.

![Graphical user interface, text, application Description automatically
generated](.//media/image7.png){width="3.8952384076990376in"
height="1.73288167104112in"}

\*Important note -- when you add or remove worker nodes in your cluster,
will need to go back to this backend pool and either add the new worker
node or remove it from this backend pool.

After entering all the worker nodes, they should all appear in attached
instances.

![Graphical user interface, application Description automatically
generated](.//media/image8.png){width="5.690476815398076in"
height="1.3289938757655293in"}

Create Front-end Listener
-------------------------

Under front-end listeners, click on **New listener**.

![Background pattern Description automatically generated with medium
confidence](.//media/image9.png){width="6.5in" height="1.0625in"}

On the next screen, enter the node port from your service ( in the
example above 30280 ), select your backend pool, and then enter 15000 as
the maximum connections, and then click save.

![Graphical user interface, application Description automatically
generated](.//media/image10.png){width="3.8701049868766404in"
height="5.414285870516186in"}

Create the load balancer
------------------------

On the panel on the right hand side of the screen, click on Create Load
Balancer.

![Graphical user interface, application, Teams Description automatically
generated](.//media/image11.png){width="2.328378171478565in"
height="3.338096019247594in"}

Get the load balancer host name
-------------------------------

The load balancer will take a few minutes to create. After it is
created, you can view the hostname from the overview page.

![Graphical user interface, application Description automatically
generated](.//media/image12.png){width="4.647619203849519in"
height="1.580488845144357in"}

Test the load balancer
----------------------

If you are following the example in this document, you can curl the
hostname and it should tell you "Hello Openshift"

![](.//media/image13.png){width="4.495238407699038in"
height="0.3683606736657918in"}

Note, it can take a couple of minutes for DNS to be updated. If you want
to test right away run

host \<hostname\>

This will return 2 IPs, like below that you can then curl instead of the
hostname if you want to test right away.

![](.//media/image14.png){width="4.471428258967629in"
height="0.45144247594050746in"}
