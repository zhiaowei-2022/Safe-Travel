<template>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous"/>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-datepicker/1.4.1/css/bootstrap-datepicker3.css"/>    
    <div class="Favourite">
    
        <div v-if="user">
            <div> <h1> {{user.displayName}}'s Favourites </h1> </div>
            <div class="container" id="Infocontainer" >
                <div>
                <h3> Favourite Tourist Attraction </h3>
                <div class="row" v-for="row in favouriteTA" :key="row" 
                    v-on:click="openModal(row.Name, row.ImageURL, row.Rating, row.Address, row.Contact, row.Description, row.Website)">
                    
                    <div class="col-4 imginput">
                    <figure >
                        <img
                        :id="row.Name"
                        :src="row.ImageURL"
                        :alt="row.Name"
                        
                        style="width:100%"
                        />
                    </figure>
                    </div>
                    <div class="col-8 favouritesModal">
                        <h4><strong> {{row.Name}} </strong></h4>
                        <br/>
                        <br/>
                        <h5> Category: {{row.Category}} </h5>
                    </div>
                    
                </div>
                <br>
                <h3> Favourite Food and Dining </h3>
                <div class="row" v-for="row in favouriteFB" :key="row" 
                    v-on:click="openModal(row.Name, row.ImageURL, row.Rating, row.Address, row.Contact, row.Description, row.Website)">
                    <div class="col-4 imginput">
                    <figure >
                        <img
                        :id="row.Name"
                        :src="row.ImageURL"
                        :alt="row.Name"
                        
                        style="width:100%"
                        />
                    </figure>
                    </div>
                    <div class="col-8 favouritesModal">
                        <h4><strong> {{row.Name}} </strong></h4>
                        <br/>
                        <br/>
                        <h5>Category: {{row.Category}} </h5>
                    </div>
                    <br>
                </div>
                </div>
            </div>
        </div>
    </div>
    <div id="searchResult" class="modal">
    <!-- Modal content -->
    <div class="modal-content">
      <span class="close" v-on:click="closeModal()">&times;</span>
      <div class="container">
        <div >
          <div id="photo">
            <!-- img -->
          </div>
        </div>
        <div class="row" style="background-color:transparent">
            <div class="col-8"></div>
            <div class="col-4"  style="text-align:right">
                <button class="btn btn-primary" id="favbut">Remove from favourites</button>
            </div>
        </div>
        <div>
          <div id="resultinfo"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import firebaseApp from "../firebase.js";
import { getAuth, onAuthStateChanged } from "firebase/auth";
import { getFirestore, collection, getDocs, setDoc, doc, deleteDoc} from "firebase/firestore"

const db = getFirestore(firebaseApp);

export default {
    name:"FavouritePage",
    methods: {
        openModal(name, imageURL, rating, address, contact, desc, web) {
            const fbuser = getAuth().currentUser.email;            
            var modal = document.getElementById("searchResult");
            var photoinfo = document.getElementById("photo");
            photoinfo.innerHTML = "<img src='" + imageURL + " 'style='width:100%;height:400px;border-radius: 30px;padding:10px'>";
            var favbut = document.getElementById("favbut");
            
            for(var index = 0; index < this.favourites.length; index++) {
                    console.log(this.favourites[index]["Name"] == name)
                    if (this.favourites[index]["Name"] == name) {
                            createDelBut(name,this.favourites);
                            
                            break;
                        } else {
                            createAddBut(name, this.favourites)
                            
                        }
            }

            function createDelBut(name,favourites) {
                favbut.innerHTML = "Remove from Favourites"
                favbut.onclick = function () {
                    removeFav(name,favourites)
                    console.log("removed")
                    console.log(favourites)
                    createAddBut(name,favourites)
                } 
                
            }
            function createAddBut(name,favourites) {
                favbut.innerHTML = "Add to Favourites"
                favbut.onclick = function () {
                    //console.log(name)
                    console.log(this.database)
                    addFav(name,favourites)
                    console.log("Added")
                    createDelBut(name,favourites)
                }     
            }

            async function removeFav(name,favourites){
                var itemname = name
                console.log("Removing Favourites: ", itemname)
                await deleteDoc(doc(db, "Users/"+String(fbuser)+"/Favourites", itemname));
                console.log("Document removed")
                console.log(favourites)
                
            }

            async function addFav(name,favourites) {
                console.log(favourites)
                try {
                    for(var i = 0; i < favourites.length; i++){
                        if(favourites[i]["Name"] == name) {
                            console.log(favourites[i])
                            const docRef = setDoc(doc(db, "Users/"+String(fbuser)+"/Favourites", name), {
                                Name: name,
                                ImageURL: favourites[i]["ImageURL"],
                                Rating: favourites[i]["Rating"],
                                Address: favourites[i]["Address"],
                                Contact: favourites[i]["Contact"],
                                Description: favourites[i]["Description"],
                                Website: favourites[i]["Website"],
                                Category: favourites[i]["Category"],
                                Overhead: favourites[i]["Overhead"]
                            })
                            console.log(docRef)
                        }
                    }
                } catch (error) {
                    console.error("Error adding document:", error)
                }
                
            }

            var resultbox = document.getElementById("resultinfo");
            resultbox.innerHTML =
                "<h4><b>" +
                name +
                "</b></h4>" +
                "<b>Rating:</b> " +
                rating +
                " / 5 <br>" +
                "<b>Address:</b> " +
                address +
                "<br>" +
                "<b>Phone:</b> " +
                contact +
                "<br><br>" +
                "<h5><b>Description:</b></h5> " +
                desc +
                "<br><br>" +
                "For more information please visit <a href='" +
                web +
                "' target='_blank' style='color:black'>here</a> <br>";
                modal.style.display = "block";
                // console.log(category)
        },
        
        closeModal() {
            var modal = document.getElementById("searchResult");
            //console.log(modal)
            modal.style.display = "none";
            window.location.reload();
        },

        async readUserFirebase() {
            this.favourites = [];
            const auth = getAuth();
            const fbuser = auth.currentUser.email;
            var z = await getDocs(collection(db,"Users/"+ String(fbuser)+"/Favourites"))
            z.forEach((doc) => {
                let row = doc.data();
                if(row["Overhead"] == "Tourist Attractions") {
                    this.favouriteTA.push(row)
                } else if (row["Overhead"] == "Food And Dining") {
                    this.favouriteFB.push(row)
                }
                this.favourites.push(row)
                this.database.push(row)
                    
            });
        }
    },

    data() {
        return {
            user:false,
            favourites: [],
            favouriteTA: [],
            favouriteFB: [],
            database: [],
        }
    },

    mounted() {
        let jquery = document.createElement("script");
        jquery.setAttribute(
        "src",
        "https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js"
        );
        document.head.appendChild(jquery);
        const auth = getAuth();
        onAuthStateChanged(auth, (user) => {
            if(user) {
                this.user = user;
                this.readUserFirebase();
            }
        })
    },
}
</script>

<style scoped>
h1, h2 {
    text-align: center;
    font-weight: bold;
    color: rgb(0, 15, 92);
}

h3 {
    text-align: left;
    margin-left: 10px;
    color:rgb(0, 15, 92);
    font-weight: bold;
}

.Favourite {
        margin-top:20px;
}

/* The Modal (background) */
.modal {
    display: none; /* Hidden by default */
    position: fixed; /* Stay in place */
    z-index: 1; /* Sit on top */
    left: 0;
    top: 0;
    width: 100%; /* Full width */
    height: 100%; /* Full height */
    /* overflow: auto; /* Enable scroll if needed */
    background-color: rgb(0,0,0); /* Fallback color */
    background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
}

/* Modal Content/Box */
.modal-content {
    background-color: #fefefe;
    margin: 15% auto; /* 15% from the top and centered */
    padding: 20px;
    border: 1px solid #888;
    width: 50%; /* Could be more or less, depending on screen size */
}

/* The Close Button */
.close {
    color: #aaa;
    text-align: right;
    padding-right:15px;
    font-size: 28px;
    font-weight: bold;
}

.close:hover,
.close:focus {
    color: black;
    text-decoration: none;
    cursor: pointer;
}

#resultinfo {
    text-align: left;
}

.favouritesModal {
    color:black;
    align-items: center;
    margin:auto;
}

.row {
    background-color: #f8f9fa;
    border-color: #f8f9fa;
    padding:10px 0px 0px 0px;
    margin: 10px 0px 0px 0px;
}

.row:hover {
    color: #212529;
    background-color: #e2e6ea;
    border-color: #dae0e5;

}

img {
    width: 100%;
    border-radius: 10px;
    object-fit: cover;
    margin:auto;
    height: 200px;
}

.btn-primary {
    background-color: lightskyblue;
    border-color: lightskyblue;
    color: black;
    font-weight: bold;
    float: right;
}
</style>
