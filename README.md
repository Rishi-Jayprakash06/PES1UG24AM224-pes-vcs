# OS Orange - 2

**Name:** Rishi J Mule  
**SRN:** PES1UG24AM224  
**Section:** D  

---

## Screenshot 1A: test_objects output showing all tests passing
![1A](screenshots/Screenshot 2026-04-21 170852.png)

## Screenshot 1B: Sharded object directory structure using find .pes/objects -type f
![1B](screenshots/Screenshot 2026-04-21 172312.png)

## Screenshot 2A: test_tree output showing all tests passing
![2A](screenshots/Screenshot 2026-04-21 170913.png)

## Screenshot 2B: Raw binary tree object inspected using xxd
![2B](screenshots/Screenshot 2026-04-21 170922.png)

## Screenshot 3A: pes init → pes add → pes status command sequence
![3A](screenshots/Screenshot 2026-04-21 170932.png)

## Screenshot 3B: cat .pes/index showing text-format staging area
![3B](screenshots/Screenshot 2026-04-21 170939.png)

## Screenshot 4A: pes log showing three commits with hashes, authors and timestamps
![4A](screenshots/Screenshot 2026-04-21 170947.png)

## Screenshot 4B: find .pes -type f showing object store growth after three commits
![4B](screenshots/Screenshot 2026-04-21 170955.png)

## Screenshot 4C: cat .pes/refs/heads/main and cat .pes/HEAD showing reference chain
![4C](screenshots/Screenshot 2026-04-21 171001.png)

## Final Test
![Final](screenshots/Screenshot 2026-04-21 171025.png)

---

## Questions and Answers

### Q5.1 — How would you implement `pes checkout <branch>`?

To implement `pes checkout`, two things must happen: updating `.pes/` metadata and restoring the working directory.

- Update `.pes/HEAD` to:

- If branch doesn’t exist, create it under `.pes/refs/heads/` with current commit hash.

To restore files:
- Read target commit → get tree hash
- Recursively traverse tree
- Write blobs to correct paths
- Delete files not present in target branch

Challenges include:
- Preventing overwrite of unsaved changes
- Handling nested directories
- Deleting obsolete files
- Ensuring atomic execution

---

### Q5.2 — How would you detect a dirty working directory conflict?

For each file in index:

1. Compare file metadata (mtime + size)
2. Compare blob hashes between:
 - current index
 - target branch

If file is:
- modified locally AND
- different in target branch

→ abort with conflict error

---

### Q5.3 — What happens in detached HEAD and how to recover?

Detached HEAD means `.pes/HEAD` stores a commit hash instead of a branch reference.

- Commits still work but are not tracked by any branch
- Switching branches makes them unreachable

Recovery:
- Create a new branch pointing to current commit
- If lost, recover using commit hash (if known)

---

### Q6.1 — Algorithm to delete unreachable objects

Use **mark-and-sweep**:

**Mark phase:**
- Start from all refs
- Traverse commits → trees → blobs
- Store in hash set

**Sweep phase:**
- Traverse `.pes/objects`
- Delete objects not in reachable set

Data structure:
- Hash table for O(1) lookup

Estimate:
- ~100,000 commits → ~2.5 million objects
- Total operations ≈ 2–5 million

---

### Q6.2 — Why GC during commit is dangerous?

During commit:
- blobs are written first
- commit object not yet created

GC might:
- see blobs as unreachable
- delete them

Result:
- commit points to missing objects → corruption

Git avoids this by:
- keeping recent objects (grace period)
- using locks (`gc.pid`)
- delaying garbage collection
