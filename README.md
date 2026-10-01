# Connecting to MongoDB on AWS EC2 with PyMongo

How I connected Python on my Mac to MongoDB running on an AWS EC2 instance.

---

## Step 1: Launch an EC2 instance

1. Go to **EC2 → Instances → Launch instances**
2. Pick **Ubuntu**
3. Choose your **Key Pair**
4. Choose your **Security Group**
5. Click **Launch instance**
6. Copy the instance's **Public IPv4 address**

---

## Step 2: SSH into the instance

```bash
ssh -i your-key.pem ubuntu@<PUBLIC-IP>
```

---

## Step 3: Install and start MongoDB

```bash
sudo apt-get install -y gnupg curl
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
```

Check it's running:

```bash
sudo systemctl status mongod
```

It should say `active (running)`.

---

## Step 4: Let MongoDB accept outside connections

```bash
sudo sed -i 's/bindIp: 127.0.0.1/bindIp: 0.0.0.0/' /etc/mongod.conf
sudo systemctl restart mongod
```

---

## Step 5: Open port 27017 in the Security Group

1. Go to **EC2 → Security Groups** and select your security group
2. Click **Edit inbound rules**
3. Click **Add rule**:
   - Type: **Custom TCP**
   - Port range: **27017**
   - Source type: **My IP**
4. Click **Save rules**

Leave the SSH rule (port 22) as it is, or you'll lock yourself out.

My inbound rules ended up as:

| Port  | Source  |
|-------|---------|
| 80    | 0.0.0.0/0 |
| 22    | My IP   |
| 27017 | My IP   |

---

## Step 6: Set up PyCharm

1. Create a new project in PyCharm (it makes a `.venv` for you)
2. Open the terminal in PyCharm and install pymongo:

```bash
.venv/bin/pip install pymongo
```

---

## Step 7: Write the Python script

Create a file called `aws.py`:

```python
import pymongo

client = pymongo.MongoClient("mongodb://<PUBLIC-IP>:27017")

db = client["demo"]
col = db["test_collection"]
col.insert_one({"success": True})

for document in col.find():
    print(document)
```

Replace `<PUBLIC-IP>` with your instance's IP.

---

## Step 8: Run it

Click the green play button in PyCharm.

Output:

```
{'_id': ObjectId('...'), 'success': True}

Process finished with exit code 0
```

exit code 0 means its fine no problems occurred

---

## Problem I hit

**`ModuleNotFoundError: No module named 'pymongo'`**

pymongo wasn't installed in the project's venv. Fixed with:

```bash
.venv/bin/pip install pymongo
```

---

##  Finishing 

When you're done: **EC2 → Instances → select instance → Instance state → Terminate instance**
