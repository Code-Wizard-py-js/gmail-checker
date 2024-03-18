<template>
  <div class="h-screen py-8 mx-20 ">
    <p class="mx-auto text-2xl text-white ">{{ processStatus.message || processStatus }}</p>
    
    <p class="mx-auto text-2xl text-white">{{ threadingData.unknon }}</p>
    <br>
    <div class="grid grid-cols-1 gap-4 sm:grid-cols-2">
      <!-- First Column (Upload File and Text Area) -->
      <div class="p-4 bg-blue-500 rounded-lg">
        <form action="" class="space-y-4">
          <h2 class="mb-4 text-xl font-semibold">Upload Leads for Validation. </h2>
          <input type="file" class="block mb-4" @change="handleFileUpload" accept=".txt">
          <textarea v-model="uploadTextareaContent" class="block w-full p-2 mb-4 border border-gray-300 rounded-md h-96" placeholder="Enter gmail-id and password format: email|pass|recovery"></textarea>
          <button @click.prevent="fetchData"  type="button" class="px-4 py-2 text-white bg-blue-800 rounded-md hover:bg-blue-600">Data Import and Setup</button>
          <button @click.prevent="clerdata"  type="button" class="px-4 py-2 ml-5 mr-8 text-white bg-blue-800 rounded-md hover:bg-blue-600">Clear Data</button>
          <button @click.prevent="checkdb"  type="button" class="px-4 py-2 ml-3 mr-3 text-white bg-blue-800 rounded-md hover:bg-blue-600">check server</button>
          <button @click.prevent="starthread"  type="button" class="px-4 py-2 ml-3 mr-3 text-white bg-blue-800 rounded-md hover:bg-blue-600">Start server</button>
          <button @click.prevent="startFetch"  type="button" class="px-4 py-2 ml-3 mr-3 text-white bg-blue-800 rounded-md hover:bg-blue-600">Start Validation</button>
          <br><br>
          <div class="flex items-end justify-end mx-auto mt-4 space-x-5">
          <button @click="downloadCSV('super')"  type="button" class="px-2 py-2 text-white bg-green-800 rounded-md hover:bg-green-600">SuperValid ({{ threadingData.supers.length }})</button>
          <button @click="downloadCSV('normal')" type="button"  class="px-2 py-2 text-white bg-gray-500 rounded-md hover:bg-blue-300">NormalValid ({{ threadingData.normal.length }})</button>
          <button @click="downloadCSV('invalid')"  type="button" class="px-2 py-2 text-white bg-red-800 rounded-md hover:bg-red-600">Invalid ({{ threadingData.invalid.length }})</button>
        </div>
          <p  class="px-3 my-4 text-white bg-gray-800 rounded-md hover:bg-red-600">{{ errotext }}</p>
        </form>
      </div>

      <!-- Second Column (Two Columns) -->
      <div class="p-4 rounded-lg bg-blue-950">
        <form action="">
          <h2 class="mb-4 text-xl font-semibold text-white">Insert gmail-id-pass for Validation. </h2>
          <textarea v-model="checkTextareaContent" class="block w-full p-5 mb-4 border border-gray-300 rounded-md h-96" placeholder="Enter gmail-id and password format: email|pass|recovery" id='hash'></textarea>
          <button @click.prevent="startValidation" class="px-4 py-2 text-white bg-blue-800 rounded-md hover:bg-blue-600">Start-Check</button>
          <button @click.prevent="copytext" class="px-4 py-2 mx-5 text-white bg-blue-800 rounded-md hover:bg-blue-600">Copy good</button>
          <p class="flex items-center justify-center text-white">{{ statuscheck }}</p>
        </form>
        <br>
        <div class="mt-4 space-x-3">
          <button @click="downloadCSV('good')" class="px-4 py-2 text-white bg-green-600 rounded-md hover:bg-green-500"> Good Emails ({{ goodEmails.length }})</button>
          <button  @click="downloadCSV('bad')" class="px-4 py-2 text-white bg-red-600 rounded-md hover:bg-red-500"> Bad Emails ({{ badEmails.length }})</button>
        </div>
      </div>
    </div>
    
    <br>
   
    <div class="grid"><button @click="logout" class="p-2 mt-4 text-white bg-blue-800 rounded">Logout</button></div>
    <br>
    <br>
  </div>
</template>

<script>
import axios from 'axios';
import { saveAs } from 'file-saver';
export default {
  name: 'Dashboard',
  data() {
    return {
      statuscheck:'',
      filedata:'',
      errotext:'',
      uploadTextareaContent: '',
      checkTextareaContent: '',
      goodEmails: [],
      badEmails: [],
      processStatus:'',
      myArray:[],
      emailarray:[],
      userId:localStorage.getItem('userid') ,
      threadingData: {
        supers: [], // Array to store super valid emails
        normal: [], // Array to store normal valid emails
        invalid: [],
        unknon:'' // Array to store invalid emails
      }
      
    };
  },
  computed: {
    uniqueSupers() {
      // Use Set to remove duplicates
      return [...new Set(this.threadingData.supers)];
    },
    uniqueNormals() {
      // Use Set to remove duplicates
      return [...new Set(this.threadingData.normal)];
    },
    uniqueInvalids() {
      // Use Set to remove duplicates
      return [...new Set(this.threadingData.invalid)];
    }
  },
  methods: {
    async startValidation() {
      await this.sendDataToServer('http://103.77.182.37:7001/api/check-mail', this.checkTextareaContent);
    },
    async startCheck() {
      await this.sendDataToServer('http://103.77.182.37:7001/api/check-mail', this.uploadTextareaContent);
    },

    startFetch() {
  // Fetch data from server using POST method
  fetch('http://127.0.0.1:3000/sse', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      body: localStorage.getItem("userid") // Pass the thread ID or any necessary data
    })
  })
    .then(response =>  response.json())
    .then(data => {
      
      // Update threadingData with new data received from the server
      this.threadingData = data;
      console.log(data)
      //this.processStatus = { message: data };
      // Call startFetch() again after a delay
      setTimeout(this.startFetch, 1000); // Adjust delay as needed
    })
    .catch(error => {
      console.error('Error fetching data:', error);
      setTimeout(this.startFetch, 1000);
    });
},
    checkdb(){
     
  const requestData = {
    userDataDir: localStorage.getItem("userid")
    
  };
  this.processStatus = {message:"wait for cookie generate "};
  // Send request to backend to process the array data
  fetch('http://127.0.0.1:3000/dataview', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json' // Set content type to JSON
    },
    body: JSON.stringify(requestData) // Convert array to JSON string before sending
  })
  .then(response => response.json())
  .then(data => {
    // Update UI with new data
    this.processStatus = data;
    // Fetch again after a delay
    //setTimeout(() => this.fetchData(), 1000); // Adjust delay as needed
  })
  .catch(error => {
    console.log('Error fetching data:', error);
    this.processStatus = {message:error};
  });


    },
    starthread() {
  const requestData = {
    thrd: localStorage.getItem("userid")
  };
  this.processStatus = { message: "wait for cookie generate" };
  // Send request to backend to process the array data
  fetch('http://127.0.0.1:3000/startthread', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json' // Set content type to JSON
    },
    body: JSON.stringify(requestData) // Convert array to JSON string before sending
  })
  .then(response => {
    if (response.ok) {
     
      this.processStatus = { message: "server Started" };
    } else {
      throw new Error('Network response was not ok.');
    }
  })
  .catch(error => {
    console.error('Error:', error);
    this.processStatus = { message: "Error starting server" };
  });
},



    copytext(){

      var emails = this.checkTextareaContent.split("\n");
var copiedText = "";

for (var i = 0; i < emails.length; i++) {
    if (emails[i].includes("Good")) {
        var emailParts = emails[i].split("|");
        if (emailParts.length > 1) {
            var emailAddresses = emailParts[1].split(" ");
            for (var j = 0; j < emailAddresses.length; j++) {
                if (emailAddresses[j].trim() !== "") { // Skip empty strings
                    copiedText += emailAddresses[j].trim();
                    if (j < emailAddresses.length - 1) {
                        copiedText += ";"; // Add semicolon and space between email addresses
                    }
                }
            }
            copiedText += "\n";
        }
    }
}

    // Copy the text to clipboard
    navigator.clipboard.writeText(copiedText).then(function() {
        console.log("Copied to clipboard: " + copiedText);
    }, function(err) {
        console.error("Failed to copy to clipboard: ", err);
    });
    },
    async sendDataToServer(url, content) {
      if (content.length>5){
      try {
        this.statuscheck="checking good/bad"
        const response = await axios.post(url, {
          emails: content.split('\n')  // Split textarea content by newline to get an array of emails
        }, {
          headers: {
            'Accept-Language': 'en-US,en;q=0.9,bn;q=0.8,fr;q=0.7',
            'Access-Control-Allow-Origin': '*',
            'Authorization': 'Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJEaXA0OSIsInJvbGUiOiJWSVAiLCJleHBpcmVkIjoiMjAyNC0wNC0wNSIsImZ1bGxOYW1lIjoiRGlwIiwiaWQiOjM0NiwiZXhwIjoxNzIzNDA1MjE0LCJpYXQiOjE3MTA0NDUyMTQsImVtYWlsIjoiZGlwZGV5MTE5QGdtYWlsLmNvbSJ9.pR73Dos8pkE8iQgta0867rAlNj3bpY3dZvxoxc28k_Xkd6U3rXIyVXA7dHbPQQ0XIJoceUeuGfvEvVcEEpFubA',
            'Connection': 'keep-alive',
            'DNT': '1',
            'Origin': 'http://testgmail.io.vn',
            'Referer': 'http://testgmail.io.vn/',
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36'
          }
        });
      
        // Update the textarea content with response emails joined by newline
        if (response.data.data && response.data.data.emails) {
          this.checkTextareaContent = response.data.data.emails.join("\n");
          
          // Separate good and bad emails
          this.goodEmails = response.data.data.emails.filter(email => email.startsWith('Good'));
          this.badEmails = response.data.data.emails.filter(email => !email.startsWith('Good'));
          this.statuscheck="All Done"
        } else {
          console.error('Invalid response format:', response.data);
        }
      } catch (error) {
        console.error('Error:', error);
        // Handle error
      }}
      else{
        
      }
    },
    

    logout() {
      this.$emit('logout');
    },
    handleFileUpload(event) {
  const file = event.target.files[0];
  const reader = new FileReader();
  this.emailarray=this.uploadTextareaContent.split('\n');
  console.log(this.emailarray)
  reader.onload = () => {
    const content = reader.result.match(/\b[\w\.-]+@[\w\.-]+\.\w{2,4}\b/g)
    
    // Send the file content to the server via Fetch API
    this.myArray.push(...content)
  };

  reader.readAsText(file);
},

fetchData() {
  if (this.myArray.length === 0) {
    alert("gmail,password is empty!");
    return; // Exit the function early
  }
  const requestData = {
    dataArray: this.myArray,
    anotherDataArray: this.emailarray,
    userid: this.userId
  };

  // Send request to backend to process the array data
  fetch('http://127.0.0.1:3000/data', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json' // Set content type to JSON
    },
    body: JSON.stringify(requestData) // Convert array to JSON string before sending
  })
  .then(response => response.json())
  .then(data => {
    // Update UI with new data
    this.processStatus = data;
    // Fetch again after a delay
    //setTimeout(() => this.fetchData(), 1000); // Adjust delay as needed
  })
  .catch(error => {
    console.log('Error fetching data:', error);
  });
}
,
clerdata(){
  const requestBody = {
  // Your request body here, for example:
  folderName: localStorage.getItem("userid")
};

fetch('http://127.0.0.1:3000/clear', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json' // Set content type to JSON
    },
    body: JSON.stringify(requestBody) // Convert array to JSON string before sending
  })
  .then(response => response.json())
  .then(data => {
    // Update UI with new data
    this.processStatus = data;
    // Fetch again after a delay
    //setTimeout(() => this.fetchData(), 1000); // Adjust delay as needed
  })
  .catch(error => {
    console.log('Error fetching data:', error);
    this.processStatus = {"message":error};
  });
},

    downloadCSV(type) {
      let emails = [];
      if (type === 'good') {
        emails = this.goodEmails;}
      if (type=="bad") {
        emails = this.badEmails;
      }
      if (type==="super"){
        emails=this.threadingData.supers
      }
      if (type==="normal"){
        emails=this.threadingData.normal
      }
      if (type==="invalid"){
        emails=this.threadingData.invalid
      }
      const csvContent = emails.join('\n');
      const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8' });
      saveAs(blob, `${type}_emails.csv`);
    }
  }
};
</script>