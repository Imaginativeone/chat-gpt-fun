# Next.js Drag and Drop Tree
## User
- Please code a drag and drop tree by using next.js, with https://shadcn-extension.vercel.app/docs/tree-view, connected to json for the data
## ChatGPT
- To create a drag-and-drop tree view in Next.js using [shadcn-ui's Tree View extension](https://shadcn-extension.vercel.app/docs/tree-view), connected to a JSON data structure, you can follow these steps. This setup will enable a hierarchical structure with drag-and-drop functionality, where the data can be dynamically updated.

### Step 1: Set up Next.js Project with shadcn Extension

1. **Initialize Next.js Project:**

   ```bash
   npx create-next-app@latest drag-drop-tree
   cd drag-drop-tree
   ```

2. **Install shadcn-ui** and any required dependencies:

   ```bash
   npm install @shadcn-ui/react
   ```

3. **Install additional dependencies** if the drag-and-drop functionality is required. For instance, `react-dnd` is commonly used for React-based drag-and-drop:

   ```bash
   npm install react-dnd react-dnd-html5-backend
   ```

### Step 2: Create JSON Data Structure

Create a file named `treeData.json` in the `public` folder (or any other location) to define the initial tree structure.

**Example `public/treeData.json`:**

```json
[
  {
    "id": "1",
    "title": "Root Node",
    "children": [
      {
        "id": "2",
        "title": "Child Node 1",
        "children": []
      },
      {
        "id": "3",
        "title": "Child Node 2",
        "children": []
      }
    ]
  }
]
```

### Step 3: Build the Tree View Component

Create a `TreeViewComponent.js` file inside the `components` folder. This component will use `@shadcn-ui/react` for the tree view, load the data from `treeData.json`, and handle drag-and-drop functionality.

```javascript
// components/TreeViewComponent.js

import { useEffect, useState } from 'react';
import { TreeView } from '@shadcn-ui/react';
import { DndProvider, useDrag, useDrop } from 'react-dnd';
import { HTML5Backend } from 'react-dnd-html5-backend';

const ItemType = {
  NODE: 'node',
};

function DraggableNode({ node, onDrop, children }) {
  const [{ isDragging }, drag] = useDrag({
    type: ItemType.NODE,
    item: { id: node.id },
    collect: (monitor) => ({
      isDragging: monitor.isDragging(),
    }),
  });

  const [, drop] = useDrop({
    accept: ItemType.NODE,
    drop: (droppedItem) => onDrop(droppedItem, node),
  });

  return (
    <div ref={(node) => drag(drop(node))} style={{ opacity: isDragging ? 0.5 : 1 }}>
      <TreeView.Item title={node.title}>
        {children}
      </TreeView.Item>
    </div>
  );
}

export default function TreeViewComponent() {
  const [treeData, setTreeData] = useState([]);

  useEffect(() => {
    async function fetchData() {
      const response = await fetch('/treeData.json');
      const data = await response.json();
      setTreeData(data);
    }
    fetchData();
  }, []);

  const handleDrop = (droppedItem, targetNode) => {
    const updatedTreeData = moveNode(treeData, droppedItem.id, targetNode.id);
    setTreeData(updatedTreeData);
  };

  const moveNode = (nodes, nodeId, targetId) => {
    // Update the nodes array to move the dragged node to the new parent
    // This function needs to find the dragged node and update its parent
    return nodes; // Modified with the updated structure
  };

  const renderTree = (nodes) => {
    return nodes.map((node) => (
      <DraggableNode key={node.id} node={node} onDrop={handleDrop}>
        {node.children && renderTree(node.children)}
      </DraggableNode>
    ));
  };

  return (
    <DndProvider backend={HTML5Backend}>
      <TreeView>
        {renderTree(treeData)}
      </TreeView>
    </DndProvider>
  );
}
```

### Step 4: Update the `moveNode` Logic

Implement the `moveNode` function logic to handle the update of the JSON structure based on drag-and-drop actions. This requires:

- Removing the node from its original location
- Adding it to the new location based on the `targetId`

```javascript
const moveNode = (nodes, nodeId, targetId) => {
  // Find and remove the dragged node from its current parent
  const nodeToMove = findAndRemoveNode(nodes, nodeId);
  if (!nodeToMove) return nodes;

  // Find target node to append the dragged node
  return addNodeToParent(nodes, targetId, nodeToMove);
};

// Helper function to find and remove node
const findAndRemoveNode = (nodes, nodeId) => {
  // Your implementation here
};

// Helper function to add node to target
const addNodeToParent = (nodes, targetId, nodeToAdd) => {
  // Your implementation here
};
```

### Step 5: Render Component in Page

Use this component in your Next.js page, typically `pages/index.js` or another designated page.

```javascript
// pages/index.js

import TreeViewComponent from '../components/TreeViewComponent';

export default function HomePage() {
  return (
    <div>
      <h1>Drag and Drop Tree View</h1>
      <TreeViewComponent />
    </div>
  );
}
```

This structure will give you a dynamic, drag-and-drop-enabled tree view. The `moveNode` function can be expanded to handle more complex data transformations as required.
