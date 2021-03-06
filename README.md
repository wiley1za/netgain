# NetGain

C# Neo4j Driver built on Neo4j 2.2 REST API including transactional support

## Current Version

[![Current Version](https://img.shields.io/nuget/v/NetGain.svg)](https://www.nuget.org/packages/NetGain/)

## Usage

	/*
  		<appSettings>
			<add key="username" value="neo4j"/>
			<add key="password" value="[[Enter your password]]]"/>
			<add key="urlRoot" value="http://localhost:7474/db/data/"/>
			<add key="defaultEncoding" value="UTF-8"/>
			<add key="defaultContentType" value="application/json"/>
		</appSettings>
	*/

	namespace NetGainConsole 
	{
		class Program
		{
			static void Main(string[] args)
			{
				NodeDemo();
				RelationshipDemo();
				Console.ReadLine();
			}

			static void NodeDemo ()
			{
				NetGain.NodeProvider provider = new NetGain.NodeProvider();
	
				Movie movie = new Movie() { tagline = "What a lovely day!", title = "Mad Max: Fury Road", year = 2015 };
	
				// Create a new node.
				NetGain.Node node = provider.Create(movie);
				Console.WriteLine(node.data.tagline);
	
				// Get the node again.
				node = provider.Get(node.id);
				Console.WriteLine(node.data.tagline);
	
				// Update the tagline.
				movie.tagline = "What a lovely day.";
				provider.Update(node.id, movie);
				node = provider.Get(node.id);
				Console.WriteLine(node.data.tagline);
	
				// Delete the node.
				provider.Delete(node.id);
	
			}
	
			static void RelationshipDemo()
			{
				NetGain.NodeProvider nodeProvider = new NetGain.NodeProvider();
				NetGain.RelationshipProvider relProvider = new NetGain.RelationshipProvider();
	
				Person person = new Person() { born = 1977, name = "Tom Hardy" };
				Movie movie = new Movie() { tagline = "What a lovely day!", title = "Mad Max: Fury Road", year = 2015 };
	
				// Create new nodes.
				NetGain.Node personNode = nodeProvider.Create(person);
				NetGain.Node movieNode = nodeProvider.Create(movie);
	
				// Create a relationship.
				dynamic relationshipProperties = new { roles = new string[] {"Mad Max"}};
				NetGain.Relationship relationship = relProvider.Create(personNode, "ACTED_IN", movieNode, relationshipProperties);
	
				// Get the relationship.
				relationship = relProvider.Get(relationship.id);
				Console.WriteLine(relationship.data.roles[0]);
	
				// Update the role.
				relProvider.SetProperty(relationship.id, "roles", new string[] { "Max Rockatansky" });
				relationship = relProvider.Get(relationship.id);
				Console.WriteLine(relationship.data.roles[0]);
	
				// Delete the relationship and nodes.
				relProvider.Delete(relationship.id);
				nodeProvider.Delete(personNode.id);
				nodeProvider.Delete(movieNode.id);
	
			}
	
		}
	}

## Getting Started

To install NetGain, run the following command in the Package Manager Console

    PM> Install-Package NetGain

To connect to Neo4j via CONFIG file:

	<appSettings>
		<add key="username" value="neo4j"/>
		<add key="password" value="[[Enter your password]]]"/>
		<add key="urlRoot" value="http://localhost:7474/db/data/"/>
		<add key="defaultEncoding" value="UTF-8"/>
		<add key="defaultContentType" value="application/json"/>
	</appSettings>

To connect to Neo4j via CODE:

	// Create a provider object.  Any class that inherits from ProviderBase will work the same.
	NetGain.LabelProvider provider = new NetGain.LabelProvider();
	provider.Credential = new System.Net.NetworkCredential("neo4j", "enter your password here");
	provider.DefaultContentType = "application/json";
	provider.DefaultEncoding = "UTF-8";
	provider.UrlRoot = "http://localhost:7474/db/data/";
	// provider.UrlEndpoint is set in each provider class.
	

### Basic Usage - Nodes, Relationships, Labels

	static void NodeDemo ()
	{
		NetGain.NodeProvider provider = new NetGain.NodeProvider();

		Movie movie = new Movie() { tagline = "What a lovely day!", title = "Mad Max: Fury Road", year = 2015 };

		// Create a new node.
		NetGain.Node node = provider.Create(movie);
		Console.WriteLine(node.data.tagline);

		// Get the node again.
		node = provider.Get(node.id);
		Console.WriteLine(node.data.tagline);

		// Update the tagline.
		movie.tagline = "What a lovely day.";
		provider.Update(node.id, movie);
		node = provider.Get(node.id);
		Console.WriteLine(node.data.tagline);

		// Delete the node.
		provider.Delete(node.id);

	}

	static void RelationshipDemo()
	{
		NetGain.NodeProvider nodeProvider = new NetGain.NodeProvider();
		NetGain.RelationshipProvider relProvider = new NetGain.RelationshipProvider();

		Person person = new Person() { born = 1977, name = "Tom Hardy" };
		Movie movie = new Movie() { tagline = "What a lovely day!", title = "Mad Max: Fury Road", year = 2015 };

		// Create new nodes.
		NetGain.Node personNode = nodeProvider.Create(person);
		NetGain.Node movieNode = nodeProvider.Create(movie);

		// Create a relationship.
		dynamic relationshipProperties = new { roles = new string[] {"Mad Max"}};
		NetGain.Relationship relationship = relProvider.Create(personNode, "ACTED_IN", movieNode, relationshipProperties);

		// Get the relationship.
		relationship = relProvider.Get(relationship.id);
		Console.WriteLine(relationship.data.roles[0]);

		// Update the role.
		relProvider.SetProperty(relationship.id, "roles", new string[] { "Max Rockatansky" });
		relationship = relProvider.Get(relationship.id);
		Console.WriteLine(relationship.data.roles[0]);

		// Delete the relationship and nodes.
		relProvider.Delete(relationship.id);
		nodeProvider.Delete(personNode.id);
		nodeProvider.Delete(movieNode.id);
	}

	static void LabelDemo ()
	{
		NetGain.LabelProvider provider = new NetGain.LabelProvider();
		// Display all the labels defined in the database.
		foreach (var label in provider.Get())
		{
			Console.WriteLine(label);
		}
	}

### Using Transactions

	static void Statement()
	{
		TransactionManager txMgr = new TransactionManager();
		Statement stmt = new NetGain.Transaction.Statement();
		stmt.statement = "Match (n) return n";
		Transaction tx = txMgr.Begin(new Statement[] { stmt });
		foreach (dynamic obj in tx.results[0].data)
		{
			try
			{
				Console.WriteLine("{0} born {1}", obj.row[0].name, obj.row[0].born);
			}
			catch (Exception ex)
			{
				Console.WriteLine(ex.Message);
			}
		}
	}

	static void StatementWithParams()
	{
		TransactionManager txMgr = new TransactionManager();
		Statement stmt = new NetGain.Transaction.Statement();
		stmt.statement = "CREATE (n {props}) return n";
		Parameter parm = new Parameter();
		parm.props = new List<dynamic>() { new { name = "B the Dogg", born = 1971}};
		stmt.parameters = parm;
		Transaction tx = txMgr.Begin(new Statement[] { stmt });
		foreach (dynamic obj in tx.results[0].data)
		{
			try
			{
				Console.WriteLine("{0} born {1}", obj.row[0].name, obj.row[0].born);
			}
			catch (Exception ex)
			{
				Console.WriteLine(ex.Message);
			}
		}
	}

	static void Commit()
	{
		TransactionManager txMgr = new TransactionManager();
		Transaction tx = txMgr.Begin();
		tx = txMgr.Commit(tx);
		Console.WriteLine(tx.expires);
	}

	static void Graph()
	{
		TransactionManager txMgr = new TransactionManager();
		Statement stmt = new NetGain.Transaction.Statement();
		stmt.statement = "CREATE ( bike:Bike { weight: 10 } ) CREATE ( frontWheel:Wheel { spokes: 3 } ) CREATE ( backWheel:Wheel { spokes: 32 } ) CREATE p1 = (bike)-[:HAS { position: 1 } ]->(frontWheel) CREATE p2 = (bike)-[:HAS { position: 2 } ]->(backWheel) RETURN bike, p1, p2";
		stmt.statement = "MATCH (n)-[r]-() return n, r";
		Transaction tx = txMgr.ExecuteGraph(new Statement[] { stmt });
	}

	static void Rest()
	{
		TransactionManager txMgr = new TransactionManager();
		Transaction tx = txMgr.Begin();
		Statement stmt = new NetGain.Transaction.Statement();
		stmt.statement = "CREATE (n) RETURN n";
		tx = txMgr.ExecuteRestReturn(tx, new Statement[] { stmt });
		Console.WriteLine(tx.results[0].data.ElementAt(0).rest.ElementAt(0).labels);
	}

	static void Errors()
	{
		TransactionManager txMgr = new TransactionManager();
		Transaction tx = txMgr.Begin();
		Statement stmt = new NetGain.Transaction.Statement();
		stmt.statement = "invalid statement";
		tx = txMgr.ExecuteRestReturn(tx, new Statement[] { stmt });
		Console.WriteLine(tx.errors.Count());
	}

### Neo4j version support

| **Version** | **Tested**  |
|-------------|-------------|
| <= 2.1      |   No        |
| 2.2         |   Yes       |

## Getting Help

[![StackOverflow](https://img.shields.io/badge/StackOverflow-Ask%20a%20question!-blue.svg)](http://stackoverflow.com/questions/ask?tags=neo4j+netgain+C%23)

[![Post an issue](https://img.shields.io/badge/Bug%3F-Post%20an%20issue!-blue.svg)](https://github.com/therealcodesailor/netgain/issues/new)

OPTIONAL IF YOU USE GITTER:
[![Gitter](https://img.shields.io/badge/Gitter-Join%20our%20chat!-blue.svg)](https://gitter.im/therealcodesailor/netgain?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

# What is Neo4j?

Neo4j is a transactional, open-source graph database.  A graph database manages data in a connected data structure, capable of  representing any kind of data in a very accessible way.  Information is stored in nodes and relationships connecting them, both of which can have arbitrary properties.  To learn more visit [What is a Graph Database?](http://neo4j.com/developer/graph-database/)
