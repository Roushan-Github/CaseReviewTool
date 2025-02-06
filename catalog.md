# Catalog Index Build

Follow these steps to successfully create Catalog Index.

## 1. Create and Publish an Item
- Navigate to **Catalog Management** → **Products** → **Create an Item**.
- Ensure the item is **published**.

## 2. Configure Category Statuses
- Navigate to **Catalog Management** → **Products** → **Categories** → **Additional Category Attributes**.
- Create two category statuses:
  - **Category Status** = 3000 → **Short Description** = Published
  - **Category Status** = 2000 → **Short Description** = Held (Unpublished)

## 3. Configure Catalog Index Build
- Navigate to **Application Platform** → **Process Modeling** → **General** → **Transactions** → **Catalog Index Build**.

## 4. Configure the Agent for First-Time Execution
- In **Catalog Index Build**, go to **Time Triggered** and configure the agent as follows:

### **Agent Criteria Details → Runtime Properties**
- **Agent Server**: Create a new server and select it (e.g., `catalogAgent`).
- **Alert Queue Name**: `DEFAULT (DEFAULT)`.
- **JMS Queue Name**: Provide your queue name (e.g., `DEV.QUEUE.1` for DTK).
- **Number of Threads**: `1`.
- **Initial Context Factory**: `File`.
- **Connection Factory**: `AGENT_QCF`.
- **Provider URL**: `file:/<path-to-jndi>` (e.g., `file:/home/cdw/DTK/devtoolkit_docker/jndi`).
- **Enable JMS Security** and **Schedule Triggered Message** (tick checkboxes).
- **Schedule Trigger Message Interval (Min.)**: `1`.

### **Agent Criteria Details → Criteria Parameter**
For first-time execution, keep default values except for the following:
1. **Collect Pending Jobs**: `Y`
2. **Organization Code**: Enter your organization name (e.g., `rkOrg`).
3. **Number of Messages**: `200`
4. **IncrementalBuildFlag**: `N`

Save the configuration.

## 5. Configure Search Index Root Directory
- Navigate to **SMA Console**.
- Search for property **yfs.searchIndex.rootDirectory**.
- Set its value to: `/var/oms/catalog`.

## 6. Create Catalog and Categories
- Navigate to **SBC** → Select your organization → **Products** → **Manage Catalogs**.
- **Create Catalog** → **Create Category** → **Create Category Again**.
- Change **Status** to `Published`.
- Add items for which you want to build the catalog index.
- Save the changes.

## 7. Build the Item Catalog Index
- Navigate to **Products** → **Manage Item Catalog Indexes**.
- **Create Index** → Select your category.
- The status will be **In Progress**.
- Validate by checking `yfs_search_index_trigger` table:
  - **01**: Scheduled
  - **02**: In Process
  - **03**: Completed
  - **04**: Active
  - **05**: Error
  - **06**: Failure

## 8. Run Catalog Index Build Agent
- Access the **DTK om-runtime container** and run the agent:
  ```sh
  cd /opt/SSFS_9.5/runtime/bin
  ./agentserver.sh <catalog_search_index_agent_server_name>
  ```

## 9. Verify Index Completion
- Navigate to **Products** → **Manage Item Catalog Indexes**.
- Wait for the status to become **Complete**.

## 10. Activate the Catalog Index
- Select your **Category ID** → Click on **Activate**.
- Now you have an **Active Lucene Index**.

## 11. Restart the App Server
- Restart the application server for the search index to reflect in the application:
  ```sh
  ./om-compose.sh restart appserver
  ```
- Wait **2-3 minutes** before accessing the application.

## 12. Validate in Call Center
- Open **Call Center** → **Product Section**.
- Select your **Organization**.
- You should now see your categories and can smoothly search items.

## 13. Alternative: Use API
- Call the `searchCatalogIndexBuild` API from **API Tester** and check the output.

## Incremental Build
- To perform an **Incremental Build**:
  - Navigate to **Agent Criteria Details** → **Criteria Parameter**.
  - Set `IncrementalBuildFlag` to `Y`.

### **Why Incremental Build?**
- When a new item is added to an existing category, an **incremental build** ensures the newly created item is reflected in the catalog.
- Steps:
  1. Add an item to an existing category.
  2. Navigate to **Products** → **Manage Item Catalog Indexes**.
  3. Select the index and click **Incremental Build**.
  4. Ensure the **catalog agent** is up and running.

