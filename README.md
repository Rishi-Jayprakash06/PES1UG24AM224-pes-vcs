# OS Orange - 2

**Name:** Rishi J Mule  
**SRN:** PES1UG24AM224  
**Section:** D  

---

## Screenshot 1A: test_objects output showing all tests passing
![1A](screenshots/1A.png)

## Screenshot 1B: Sharded object directory structure using find .pes/objects -type f
![1B](screenshots/1B.png)

## Screenshot 2A: test_tree output showing all tests passing
![2A](screenshots/2A.png)

## Screenshot 2B: Raw binary tree object inspected using xxd
![2B](screenshots/2B.png)

## Screenshot 3A: pes init → pes add → pes status command sequence
![3A](screenshots/3A.png)

## Screenshot 3B: cat .pes/index showing text-format staging area
![3B](screenshots/3B.png)

## Screenshot 4A: pes log showing three commits with hashes, authors and timestamps
![4A](screenshots/4A.png)

## Screenshot 4B: find .pes -type f showing object store growth after three commits
![4B](screenshots/4B.png)

## Screenshot 4C: cat .pes/refs/heads/main and cat .pes/HEAD showing reference chain
![4C](screenshots/4C.png)

## Final Test
![Final](screenshots/final.png)

---

## Questions and Answers

---

### Q5.1 — How would you implement pes checkout <branch>?

To implement pes checkout, two things must happen: the `.pes/` directory must be updated, and the working directory must be restored to match the target branch.

For `.pes/` updates:
- Update HEAD to:
  `ref: refs/heads/<branchname>`
- If the branch does not exist, create a file under `.pes/refs/heads/` containing the current commit hash.

For restoring the working directory:
- Read the target branch’s commit object and get its tree hash  
- Recursively traverse the tree  
- Write each blob to its correct path  
- Delete files not present in the target branch  

Challenges:
- Handling dirty working directory (avoid overwriting changes)  
- Recursive directory traversal  
- Proper deletion of files  
- Ensuring atomic execution  

---

### Q5.2 — How would you detect a dirty working directory conflict using only the index and object store?

For each file in the index:

1. Check metadata:
   - Compare file mtime and size with index values  
   - If different → file is modified  

2. Check content:
   - Get blob hash from index  
   - Get blob hash from target branch  

Decision:
- If file is modified locally AND differs between branches → abort checkout  
- If file differs between branches but matches index → safe to overwrite  

This avoids reading full file contents.

---

### Q5.3 — What happens if you make commits in detached HEAD state, and how do you recover?

Detached HEAD means `.pes/HEAD` contains a commit hash instead of a branch reference.

Behavior:
- Commits are created normally  
- No branch tracks them  
- Switching branches makes them unreachable  

Recovery:
- Before switching: create a new branch pointing to current commit  
- After switching: recover using commit hash (if known)  

---

### Q6.1 — Describe an algorithm to find and delete unreachable objects.

Use a mark-and-sweep algorithm.

Mark phase:
- Start from all refs in `.pes/refs/heads/` and HEAD  
- Traverse commits → trees → blobs  
- Store all reachable object hashes  

Sweep phase:
- Traverse `.pes/objects/`  
- Reconstruct object hash from file path  
- Delete objects not in reachable set  

Data structure:
- Hash table for O(1) lookup  

Estimate:
- ~100,000 commits → ~2.5 million objects  
- Total operations ≈ 2–5 million  

---

### Q6.2 — Why is it dangerous to run GC concurrently with a commit?

During commit:
- Blob objects are written first  
- Commit object is written later  

Race condition:
- GC may see blobs as unreachable  
- Delete them  
- Commit later references missing blobs → corruption  

Git avoids this by:
- Using a grace period (recent objects not deleted)  
- Using lock files (`gc.pid`)  
- Delaying garbage collection  
