# Some thoughts about rebuilding TiDB Cloud console UI

I really love the DuckDB UI https://motherduck.com/docs/getting-started/interfaces/motherduck-quick-tour/ Source code: https://github.com/duckdb/duckdb-ui. DuckDB UI is very editor (notebook) centric. Other features are mostly hiding the "Settings". I like this a lot. But for TiDB Cloud, now we do have features like Monitoring (dashboards), Networking and the generic hierarchy of Org > Projects > Clusters management.  

**Why DuckDB chose editor-centric design**

DuckDB's context makes this choice almost inevitable. DuckDB is an embedded analytical database — there's no server to manage, no networking to configure, no clusters to scale. The entire operational surface area that TiDB Cloud needs (monitoring, networking, org/project/cluster hierarchy) simply doesn't exist for DuckDB. When you strip away all the infrastructure management, what's left? The query. So the notebook becomes the entire product.

But I think there's a deeper strategic insight beyond just "they don't have infra to manage." MotherDuck (the cloud service on top of DuckDB) *does* have settings, billing, access tokens, regions — you can see it in your second screenshot. But they deliberately pushed all of that to the periphery. The bet they're making is that **the query editor is where value is created**, and everything else is overhead that should be minimized. This mirrors what Jupyter did for data science — nobody opens Jupyter to check cluster health.

**The AI/agent angle makes this even more interesting**

Here's where it gets relevant to TiDB Cloud's positioning. If AI agents become primary database users, the interface bifurcates into two very different surfaces:

For **human users**, an editor-centric design makes increasing sense because AI assistants will handle the operational complexity. Why would a developer manually navigate through Monitoring > Dashboard > Slow Queries when they can type "show me the slowest queries from the last hour" in a notebook cell? The notebook becomes the universal interface — you write SQL, you ask the AI questions about your cluster, you get back both data results and operational insights in the same place.

For **agent users**, they don't need a UI at all — they need APIs and MCP servers (which you've already been exploring for TiDB Cloud). The agent's "interface" is the control plane and data plane APIs.

**The tension for TiDB Cloud specifically**

The challenge is that TiDB Cloud genuinely has operational complexity that DuckDB doesn't. You can't just hide cluster scaling, network configuration, and monitoring behind a settings page — these are core workflows for your users, especially DBAs and platform teams. A pure notebook-centric approach would underserve those personas.

But I think the interesting strategic question isn't "should we copy DuckDB's design" — it's **"what if the notebook absorbed the operational workflows too?"** Imagine a TiDB Cloud UI where:

- `SHOW CLUSTER STATUS` returns a live monitoring widget inline
- You can run `ALTER CLUSTER SET scaling_policy = 'auto'` right from the editor
- The AI assistant can say "I notice your RU consumption spiked 3x in the last hour, here's the query causing it" proactively in the notebook context
- Network rules, backup policies, and even org management become SQL-like commands or natural language instructions

This would be the "Cloudflare of the AI era" angle applied to the UI — make the database itself the control plane, with the editor as the single surface. DuckDB can do this trivially because they have no infra. TiDB Cloud would need to deliberately *flatten* infra management into the query interface, which is harder but potentially more powerful.

**My take:** DuckDB's design is correct for the direction the industry is heading, but for TiDB Cloud, the path isn't to hide operational features in settings — it's to **promote them into the editor**. The notebook should be the place where you both query data *and* manage your database, with AI as the bridge between natural language intent and the underlying operations.
