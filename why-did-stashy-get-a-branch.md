# TIL: Stashy Growth Story 🌱🥸

Today I learned that a stash doesn’t need to stay buried forever.  
With a little mustache magic, even the shyest stash can grow into a proud branch.  

---

## The Story of Stashy

Once upon a repo, there lived a scrappy little stash named **Stashy**.  
He carried half-baked fixes, forgotten experiments, and late-night “I’ll clean this later” ideas.  
But poor Stashy was never allowed to shine — always trapped in `git stash list`,  
waiting, forgotten.  

One day, his suave cousin **Mustash** 🥸 appeared.  
Mustash was polished, confident, and curled at the edges with style.  
He looked at Stashy and said:  

> “Why hide in the shadows? With a little magic,  
>  you can grow into something more… a Branch.”  

Together, they cast a spell (well… more like a bash script):  

```
#!/bin/bash

# Script to create branch references directly from Git stashes (no checkout/apply)

echo "Creating branch references from Git stashes..."
echo "============================================="

# First, let's see what stashes we have
echo "Current stashes:"
git stash list

echo ""

# Get the number of stashes
stash_count=$(git stash list | wc -l)

if [ $stash_count -eq 0 ]; then
    echo "No stashes found!"
    exit 1
fi

echo "Creating branch references with commit ID suffixes (no checkout, no changes to working directory)..."

# Loop through each stash and create a branch reference
for i in $(seq 0 $((stash_count - 1))); do
    stash_name="stash@{$i}"
    base_branch_name="stash_$((i + 1))"
    
    # Get short commit ID for the stash
    stash_commit_id=$(git show --format="%h" -s "$stash_name")
    
    # Always use commit ID suffix
    branch_name="${base_branch_name}_${stash_commit_id}"
    
    # Check if stash exists
    if git stash list | grep -q "$stash_name"; then
        
        # Check if branch already exists (shouldn't happen with commit ID, but just in case)
        if git show-ref --verify --quiet refs/heads/"$branch_name"; then
            echo "⏭ Branch '$branch_name' already exists, skipping"
            continue
        fi
        
        echo "Creating branch '$branch_name' pointing to '$stash_name'"
        
        # Create branch reference directly to stash commit
        git branch "$branch_name" "$stash_name"
        
        echo "✓ Branch '$branch_name' created"
    else
        echo "⚠ Stash '$stash_name' not found"
    fi
done

echo ""
echo "Summary of created branches:"
git branch | grep "stash_"

echo ""
echo "Branch references created. Stashes are preserved."
echo "To view stash content: git show stash_1"
echo "To work on a stash: git checkout stash_1"
```

And just like that, **Branch** was born.  
Every stash became a neat new branch with its own name:  
`stash_1_xxx`, `stash_2_yyy`, and so on.  

No longer forgotten, Stashy stood tall as a branch,  
while Mustash twirled his tips and whispered:  

> “A stash without a branch is just forgotten code,  
>  and a mouth without a mustache is just unfinished business.”  

---

## Commands

- Peek at Stashy’s new branch:  
  git show stash_1  

- Work on it directly:  
  git checkout stash_1  

---

✅ **TIL:** With the right script, your stashes don’t have to stay hidden.  
They can grow into branches — with style, thanks to Mustash.  
