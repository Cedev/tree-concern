# tree-concern

A tree concern for rails models.
The `Tree` concern adds `:parent` and `:children` relationships and validates that these remain a tree.
It provides multiple ways of querying ancestors and descendants.

## Usage

Add `tree_concern` to your application's `Gemfile`:

```
gem "tree_concern", github: "Cedev/tree_concern"
```

And run `bundle`

### Models

Add `include TreeConcern::Tree` to a rails model.

#### Example usage

This simple model is a `Tree` where each node in the tree has a `:name`. It was generated by running `rails generate scaffold Node name:string`

```
class Node < ActiveRecord::Base
  include TreeConcern::Tree
  
  def to_s
    self.name + '.' + self.id.to_s
  end
end
```

### Migrations

Add `include TreeConcern::Migration` to a db migration, and `add_tree :table_name` to add the `Tree`-specific columns to a table.
This will add a "parent_id" column to the table, along with an appropriate foreign key.

The included `add_tree` will make a monomorphic relationship. Polymorphism of tree nodes is not supported.

#### Example migration

```
class CreateNodes < ActiveRecord::Migration
  include TreeConcern::Migration
  
  def change
    create_table :nodes do |t|
      t.string :name
      t.timestamps null: false
    end
    add_tree :nodes
  end
end
```

### Controllers

The `:parent` relationship is represented by a foreign key named `"parent_id"`.
In order to allow the parent to be changed add `:parent_id` to the permitted parameters (`params.permit`) in your controller.

#### Example controller

```
class NodesController < ApplicationController
  # ...

  private
    # ...

    # Never trust parameters from the scary internet, only allow the white list through.
    def node_params
      params.require(:node).permit(:name, :parent_id)
    end
end
```

### Views

Forms should submit new parents for tree nodes by submitting the `:id` of the parent as `"parent_id"`

#### Example dropdown

```
  <div class="field">
    <%= f.label :parent %><br>
    <%= f.collection_select :parent_id, Node.order(:name),:id,:name, include_blank: true %>
  </div>
```

## Querying

The `Tree` concern adds multiple ways of querying ancestors and descendants. `TreeConcern` query can add these to any class with a `:parent` and `:children`.

#### `:parent`
The parent of the node.

#### `:children`
Direct children of the node. These are the other instances that have this node as their `:parent`.

#### `:ancestors`
Ancestors of a node, closest ancestor first. The ancestors are the node's parent, it's parent's parent, etc. If `a.ancestor_of?(b)` then `a` will come after `b` in the results.

#### `:supertrees`
The node and all its ancestors, closest ancestor first. Includes the node, the node's parent, it's parent's parent, etc. If `a.ancestor_of?(b)` then `a` will come after `b` in the results.

#### `:path`
Supertrees of a node, root first. This is the path from the root to the node, including the node.

#### `:parent_path`
Path to the node's parent, root first. This is the path from the root to the node's parent, or empty if the node is root.

#### `:root`
The root of the tree that this node is a subtree of. Same as `supertrees.last`.

#### `:subtrees`
The node and all of its descendants. Includes the node, the node's children, it's chidrens' children, etc. If `a.descendant_of(b)` then `a` will come after `b` in the results.

#### `:depth_first`
The node and all of its descendants, depth first. 

#### `:breadth_first`
The node and all of its descendants, breadth first.

#### `:post_order`
The node and all of its descendants, descendants first. If `a.descendant_of(b)` then `a` will come **before** `b` in the results.

#### `:descendants`
All of a node's descendants, depth first. The descendants are the node's children, it's childrens' children, etc. If `a.descendant_of(b)` then `a` will come after `b` in the results.

#### `:descendants_depth_first`
Descendants of a node, depth first. 

#### `:descendants_breadth_first`
Descendants of a node, breadth first.

#### `:descendants_post_order`
Descendants of a node, descendants first. If `a.descendant_of(b)` then `a` will come **before** `b` in the results.

#### `:ancestor_of?(other)`
`>` for trees. `false` if they're the same. `false` if they can't be compared because neither is a subtree of the other. Same as `in?(other.ancestors)`.

#### `:descendant_of?(other)`
`<` for trees. `false` if they're the same. `false` if they can't be compared because neither is a subtree of the other. Same as `ancestors.include?(other)`.

#### `:supertree_of?(other)`
`≥` for trees. `true` if they're the same. `false` if they can't be compared because neither is a subtree of the other. Same as `in?(other.supertrees)`.

#### `:subtree_of?(other)`
`≤` for trees. `true` if they're the same. `false` if they can't be compared because neither is a subtree of the other. Same as `supertrees.include?(other)`.

#### `:child_of?(other)`
Same as `parent == other`.

#### `:parent_of?(other)`
Same as `== other.parent`.

#### `:root_of?(other)`
Same as `== other.root`.

#### `:root?`
Is the root of the tree (doesn't have a parent, isn't a child).

#### `:child?`
Is the child of another node (has a parent, isn't a root).

#### `:parent?`
Is the parent of another node (has any children). Same as `children.any?`
