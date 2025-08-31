# type-book
find your type ✨️ 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Personality Connect Ultimate</title>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
<style>
body { font-family: Arial; margin:0; background: linear-gradient(135deg,#f6d365,#fda085); background-size:400% 400%; animation:bgAnim 15s ease infinite; display:flex; justify-content:center; align-items:center; height:100vh;}
@keyframes bgAnim {0%{background-position:0% 50%;}50%{background-position:100% 50%;}100%{background-position:0% 50%;}}
#container { width:350px; max-width:100%; height:600px; position:relative; background:white; border-radius:15px; box-shadow:0 4px 20px rgba(0,0,0,0.2); padding:20px; }
.card { position:absolute; width:100%; height:100%; border-radius:15px; box-shadow:0 4px 15px rgba(0,0,0,0.2); display:flex; flex-direction:column; justify-content:center; align-items:center; transition: transform 0.3s ease, opacity 0.3s ease;}
.card img { width:100px; height:100px; border-radius:50%; margin-bottom:10px;}
.card h3{margin:5px 0;} .card p{font-size:14px;text-align:center;}
button{position:absolute;bottom:-60px;width:120px;padding:10px;border:none;border-radius:10px;cursor:pointer;}
#likeBtn{left:0;background:green;color:white;}
#dislikeBtn{right:0;background:red;color:white;}
</style>
</head>
<body>

<div id="container"></div>
<button id="likeBtn">LIKE</button>
<button id="dislikeBtn">DISLIKE</button>

<script>
const firebaseConfig = { apiKey:"YOUR_API_KEY", authDomain:"YOUR_PROJECT.firebaseapp.com", projectId:"YOUR_PROJECT_ID", storageBucket:"YOUR_PROJECT.appspot.com", messagingSenderId:"SENDER_ID", appId:"APP_ID" };
firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.firestore();

let users=[], currentIndex=0, currentUserId=null;
let currentUser={hobby:"game", idol:"elon musk", history:"ethiopia", type:"adventurous", personality:"Adventurous Gamer 🎮"}; 

// Sign in anonymously for testing
auth.signInAnonymously().then(res=>{ currentUserId=res.user.uid; loadUsers(); });

function loadUsers(){
    db.collection("users").get().then(snapshot=>{
        users=snapshot.docs.map(doc=>({id:doc.id, ...doc.data()}))
                 .filter(u=>u.id!==currentUserId);
        users.sort((a,b)=> aiMatchScore(b)-aiMatchScore(a));
        showCard();
    });
}

function aiMatchScore(user){
    let score=0;
    if(user.hobby===currentUser.hobby) score++;
    if(user.idol===currentUser.idol) score++;
    if(user.history===currentUser.history) score++;
    if(user.type===currentUser.type) score++;
    if(user.personality===currentUser.personality) score++;
    return score;
}

function showCard(){
    const container=document.getElementById('container');
    container.innerHTML='';
    if(currentIndex>=users.length){ container.innerHTML="<h3>No more users</h3>"; return; }
    const u=users[currentIndex];
    const card=document.createElement('div'); card.className='card';
    card.innerHTML=`<img src='${u.avatar || "https://via.placeholder.com/100"}'><h3>${u.name}</h3><p>${u.personality}</p><p>Hobby: ${u.hobby}</p><p>Idol: ${u.idol}</p>`;
    container.appendChild(card);
}

function swipeRight(){
    db.collection("likes").add({fromUserId:currentUserId,toUserId:users[currentIndex].id,liked:true});
    checkMatch(users[currentIndex].id);
    currentIndex++; showCard();
}

function swipeLeft(){
    db.collection("likes").add({fromUserId:currentUserId,toUserId:users[currentIndex].id,liked:false});
    currentIndex++; showCard();
}

function checkMatch(toUserId){
    db.collection("likes")
      .where("fromUserId","==",toUserId)
      .where("toUserId","==",currentUserId)
      .where("liked","==",true)
      .get()
      .then(snapshot=>{
        if(!snapshot.empty){
            db.collection("matches").add({user1Id:currentUserId,user2Id:toUserId});
            alert("It's a match! 🎉");
        }
      });
}

document.getElementById('likeBtn').addEventListener('click',swipeRight);
document.getElementById('dislikeBtn').addEventListener('click',swipeLeft);
</script>

</body>
</html>
