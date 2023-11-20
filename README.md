# voting-system-react

`index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Voting System</title>
    <link rel="stylesheet" href="main.css">
</head>
<body>
    <h1>College Voting System</h1>
    <div class="container">
        <div class="user-registration"> 
            <h2>Get yourself registered!</h2>
            <input type="text" id="userIdentity" placeholder="Enter your identity">
            <input id="registerButton" type="submit" value="Register"/>
        </div>
        <div class="ballot-creation">
            <h2>Do Create Ballot</h2>
            <input type="text" id="ballotTitle" placeholder="Enter ballot title">
            <input type="text" id="ballotOptions" placeholder="Enter ballot options">
            <input id="createBallotButton" type="submit" value="Create"/>
        </div>
        <div class="voting">
            <h2>cast your Vote</h2>
            <input type="number" id="ballotid" placeholder="Enter ballot id">
            <input type="text" id="ballottitle" placeholder="Enter Your Name">
            <input type="text" id="ballotOptions" placeholder="For Whom You Want To Case Vote">
            <input id="castVoteButton" type="submit" value="Vote"/>
        </div>
        <div class="ballots">
            <h2>Get to know Ballot Details</h2>
            <ul id="ballotList"></ul>
            <input id="getBallotsButton" type="submit" value="Submit"/>
        </div>
    </div>
    <script src="index.js"></script>
</body>
</html>
```

`main.css`

```css
body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f8f9fa;
}

.container {
    max-width: 800px;
    margin: 20px auto;
    align-items: center;
    background-color: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
}

h1, h2 {
    color: #007BFF;
    text-align: center;
}

.user-registration, .ballot-creation, .voting, .ballots {
    margin-bottom: 30px;
}

input[type="text"],
input[type="number"],
input[type="submit"] {
    width: 100%;
    padding: 15px;
    margin-bottom: 15px;
    border: 1px solid #ced4da;
    border-radius: 4px;
    box-sizing: border-box;
    font-size: 16px;
}

input[type="submit"] {
    background-color: #007BFF;
    color: #fff;
    cursor: pointer;
    transition: background-color 0.3s ease;
}

input[type="submit"]:hover {
    background-color: #0056b3;
}

ul {
    list-style-type: none;
    padding: 0;
}

/* Responsive layout */
@media (max-width: 600px) {
    .container {
        padding: 15px;
    }
}

```

`index.js`

```js
import {voting_backend} from "../../declarations/voting_backend";


async function registerUser() {
  const identity = document.getElementById('userIdentity').value;
  try {
    const response = await voting_backend.registerUser(identity);
    alert(response);
  } catch (error) {
    alert(`Error registering user: ${error}`);
  }
}

async function createBallot() {
  const title = document.getElementById('ballotTitle').value;
  const options = document.getElementById('ballotOptions').value;
  try {
    const ballotId = await voting_backend.createBallot(title, options);
    alert(`New ballot created with ID: ${ballotId}`);
  } catch (error) {
    alert(`Error creating ballot: ${error}`);
  }
}

async function castVote() {
  const ballotId = parseInt(document.getElementById('ballotid').value);
  const title = document.getElementById('ballottitle').value;
  const options = document.getElementById('ballotOptions').value;
  try {
    const response = await voting_backend.castVote(ballotId, title, options);
    alert(response);
  } catch (error) {
    alert(`Error casting vote: ${error}`);
  }
}

async function displayBallots() {
  const ballotList = document.getElementById('ballotList');
  const ballots = await voting_backend.getBallots();

  ballotList.innerHTML = '';

  for (const ballot of ballots) {
    const li = document.createElement('li');
    li.textContent = `Ballot ID: ${ballot.id}, Title: ${ballot.title}, Options: ${ballot.options}, Votes: ${ballot.votes}`;
    ballotList.appendChild(li);
  }
}

document.getElementById('registerButton').addEventListener('click', function(event) {
  event.stopImmediatePropagation();
  registerUser();
});
document.getElementById('createBallotButton').addEventListener('click', function(event){
  event.stopImmediatePropagation();
  createBallot();
});
document.getElementById('castVoteButton').addEventListener('click', function(event){
  event.stopImmediatePropagation();
  castVote();
});
document.getElementById('getBallotsButton').addEventListener('click', function(event){
  event.stopImmediatePropagation();
  displayBallots();
});

 
```

`main.mo`

```js 
import Array "mo:base/Array";


actor VotingSystem {
  type List<T> = ?(T, List<T>);
  type Ballot = {
    id: Nat;
    title: Text;
    options: Text;
    votes: Nat;
  };

  type User = {
    identity: Text;
    hasVoted: Bool;
  };

  var ballots : [Ballot] = [];
  var users : [User] = [];

  

  public func registerUser(identity: Text) : async Text {
    let newUser : User = { identity = identity; hasVoted = false };
    let updatedUsers = Array.append(users, [newUser]);
    users := updatedUsers;
    "User Has Been Registered For Voting";
  };

  public func createBallot(title: Text, options: Text, ) : async Nat {
    let ballotId = Array.size(ballots);
    let newBallot : Ballot = {
      id = ballotId;
      title = title;
      options = options;
      votes = 0;
    };
    ballots := Array.append(ballots, [newBallot]);
    return ballotId;
  };

  public func castVote(ballotId: Nat, title: Text, options: Text) : async Text {
    if (ballotId >= Array.size(ballots)) {
       return "Ballot does not exist.";
    };
    let new : Ballot = {
      id = ballotId;
      title = title;
      options = options;
      votes = 1;
    };
    ballots := Array.append(ballots, [new]);
    let message = "Vote Casted Succesfully for " # options;
    return message;
  };

  public func getBallots() : async [Ballot] {
    ballots;
  };
};
```

`package.json`

```json
{
  "name": "voting_frontend",
  "version": "0.2.0",
  "description": "Internet Computer starter application",
  "keywords": [
    "Internet Computer",
    "Motoko",
    "JavaScript",
    "Canister"
  ],
  "scripts": {
    "build": "webpack",
    "prebuild": "dfx generate",
    "start": "webpack serve --mode development --env development",
    "deploy:local": "dfx deploy --network=local",
    "deploy:ic": "dfx deploy --network=ic",
    "generate": "dfx generate voting_backend"
  },
  "dependencies": {
    "@dfinity/agent": "^0.19.3",
    "@dfinity/candid": "^0.19.3",
    "@dfinity/principal": "^0.19.3"
  },
  "devDependencies": {
    "assert": "2.0.0",
    "buffer": "6.0.3",
    "copy-webpack-plugin": "^11.0.0",
    "dotenv": "^16.0.3",
    "events": "3.3.0",
    "html-webpack-plugin": "5.5.0",
    "process": "0.11.10",
    "stream-browserify": "3.0.0",
    "terser-webpack-plugin": "^5.3.3",
    "util": "0.12.4",
    "webpack": "^5.73.0",
    "webpack-cli": "^4.10.0",
    "webpack-dev-server": "^4.8.1"
  },
  "engines": {
    "node": "^12 || ^14 || ^16 || >=17",
    "npm": "^7.17 || >=8"
  },
  "browserslist": [
    "last 2 chrome version",
    "last 2 firefox version",
    "last 2 safari version",
    "last 2 edge version"
  ]
}

```
