<script>
  import { onMount } from 'svelte';
  import "carbon-components-svelte/css/all.css";
  import LibLoader from './components/LibLoadder.svelte';
  import {
    Header,
    HeaderUtilities,
    HeaderGlobalAction,
    SkipToContent,
    Content,
    Grid,
    Row,
    Column,
    Theme,
    Button,
  } from "carbon-components-svelte";
  import Main from "./pages/main.svelte";
  import Info from "./pages/info.svelte";
  import Settings from "./pages/settings.svelte";
  import SettingsAdjust from "carbon-icons-svelte/lib/SettingsAdjust.svelte";
  import Sun from "carbon-icons-svelte/lib/Sun.svelte";
  import Information from "carbon-icons-svelte/lib/Information.svelte";
  import nid from "./modules/nid/nid";

  const pluginname = "Apple Watch Hub";
  const pluginemoji = "⌚️";
  var parent = "";
  let PreLoadedList = [];
  let plugin;
  let PlugiAapiUrl = "https://plugins.nomie.app/v1/nomie-plugin.js";
  
  plugin = new NomiePlugin({
        name: pluginname,
        emoji: pluginemoji,
        description: "Apple Watch Hub Plugin to sync data to Apple Watch App",
        uses: ['selectTrackables', 'searchNotes', 'onLaunch'],
        version: "0.5",
        addToCaptureMenu: true,
        addToMoreMenu: true,
        addToWidgets: false,
      }); 
  let inNomie = false;
  let isSideNavOpen = false;
  let theme = "g10";
  let mode = "hidden";
  let loading = true;
  let view = "main";

  //storage
  let selection = "";
  let custom = [];
  let blacklist = [];
  let apiaddress = "";
  let apikey ="";
  let awapiaddress ="s4eawapi.smarter4ever.com/";
  let awapikey = "";
  let synconstart = false;
  let onlaunch = false;

  // Load init params
  function loadInitParams() {
    parent = getParentUrl();
    plugin.onUIOpened(async () => {
      mode = 'modal';
    });

    plugin.onLaunch(async () => {
      console.log("==================👻👻=================");
      console.log("⌚️ Apple Watch Plugin onLaunch");
      console.log("==================👻👻=================");
      mode = 'hidden';
      onlaunch = true;
      if (synconstart) {
      await onLaunchStart()}
      else {console.log("⌚️=> Sync on Start is not enabled")}
      

    });

    plugin.onRegistered(async () => {
      inNomie = true;
      loading = false;
      await plugin.storage.init()
      selection= await plugin.storage.getItem('selection') || [];
      custom = await plugin.storage.getItem('custom') || [];
      blacklist = await plugin.storage.getItem('blacklist') || [];
      apiaddress = await plugin.storage.getItem('apiaddress') || "https://6.nomie.app/api/capture";
      apikey = await plugin.storage.getItem('apikey') || "";
      awapikey = await plugin.storage.getItem('awapikey') || "";
      synconstart = await plugin.storage.getItem('synconstart') || false;
      

      if (plugin.prefs.theme == "light") {
        theme = "g10"}
      else if (plugin.prefs.theme == "dark") {
        theme = "g90"}  
      else {theme = "g10"} 
    })

    setTimeout(() => {
      if (loading) {
        inNomie = false;
      }
    }, 400);
  }

  // change theme
  function toggleTheme(){
    if (theme == "white"){
      theme = "g10"}
    else if (theme == "g10"){
      theme = "g80"}
    else if (theme == "g80"){
      theme = "g90"}
    else if (theme == "g90"){
      theme = "g100"}
    else {
      theme = "white"}
 }

 // Get parent
 function getParentUrl() {
    var isInIframe = (parent !== window),
        parentUrl = null;

    var parentfound = null;
    
    if (isInIframe) {
        parentUrl = document.referrer;
    }

    if (parentUrl.includes("nomie") ) {
      parentfound = "Nomie"
    }
    else {parentfound = "Smarter4Ever"}

    return parentfound;
}


//view main page
function showMain(){
  view = "main"
  window.scrollTo(0,0);
 }
 
 //view info page
 function showInformation(){
  view = "info"
  window.scrollTo(0,0);
 }

 //view settings page
 function showSettings(){
  view = "settings"
  window.scrollTo(0,0);
 }

 function saveSettings(){
  plugin.storage.setItem('apiaddress', apiaddress);
  plugin.storage.setItem('apikey', apikey);
  plugin.storage.setItem('awapikey', awapikey);
  plugin.storage.setItem('synconstart', synconstart);
  
  showMain();
 }

 async function exitSettings(){
  apiaddress = await plugin.storage.getItem('apiaddress') || "";
      apikey = await plugin.storage.getItem('apikey') || "";
      awapikey = await plugin.storage.getItem('awapikey') || "";
      synconstart = await plugin.storage.getItem('synconstart') || false; 
  showMain();
 }

 function saveTrackables(element){
  console.log("⌚️=> Save Trackable configuration");
  plugin.storage.setItem('selection', element.detail[0]);
  plugin.storage.setItem('custom', element.detail[1]);
  plugin.storage.setItem('blacklist', element.detail[2]);

 }

 async function syncTrackables(element){
  saveTrackables(element)
  console.log("⌚️=> Sync API Key");
  await pushS4EAPI();
  console.log("⌚️=> Sync Trackable configuration")

  // sync
  
  const res = await fetch("https://s4eawapi.smarter4ever.com/awsettrackable", {
			method: 'POST',
      body: JSON.stringify({"api_key":awapikey,"trackables":element.detail[3]}),
      headers: { "content-type": "application/json"}
			//body: JSON.stringify({"trackables":element.detail[2] ,"api_key": awapikey})
      //body: JSON.parse(JSON.stringify({trackables:element.detail[2] ,api_key: awapikey}))
		})
		
		const json = await res.json()
    //const json = res;
		//const result = JSON.stringify(json)
  //end sync
 }

 async function pushS4EAPI(){
  var encoded = btoa(apikey).toString('base64');
  const payload = {"api_key": awapikey,"s4eapi_key": encoded,"s4eapi_url":apiaddress};
  const res = await fetch("https://s4eawapi.smarter4ever.com/awsets4eapi", {
			method: 'POST',
      body: JSON.stringify(payload),
      headers: { "content-type": "application/json"}
		})
		
		const json = await res.json()
    //const json = res;
		//const result = JSON.stringify(json)

 }

 onMount(async () => {
  loadInitParams();
 })


// ALL ON LAUNCH CODE START
async function onLaunchStart() {
    await onLaunch_changeDiscoverySelection(); //done
    await onLaunch_updateCustomList();
    await onLaunch_updateBlackList();
    setTimeout(async ()=>{await onLaunch_SaveSyncSelection()},2000)
  }

    async function onLaunch_updateCustomList(){
  let temp = custom;
  temp.forEach(async(trackable,index)=>{
    let prefix = ""
    if (!trackable.tag.includes("#") && !trackable.tag.includes("@")) {
    prefix = "#";
    if (trackable.type == "person"){
      prefix = "@"
    } }
    let value = await plugin.getTrackable(prefix+trackable.tag);
    
    
    setTimeout(()=>{  if (trackable.type == "person"){ temp[index] = {
              id:nid(),
              username: value.id,
              name: value.trackable.person.displayName,
              emoji:"😎" ,
              displayName: value.trackable.person.displayName,
              tag: value.trackable.person.username ,
              type: "person",
              color: "",
              label: value.trackable.person.displayName,
              include: "",
              uom: "",
              min: "",
              max: "",
              defaultvalue: "",
              note: "",
              picks: "",
              
            }}
          else {temp[index] = {
              id:nid(),
              name:value.trackable.tracker.label, 
              username: value.id,
              emoji:value.trackable.tracker.emoji ,
              displayName: value.trackable.tracker.label,
              tag: value.trackable.tracker.tag ,
              type: value.trackable.tracker.type,
              color: value.trackable.tracker.color,
              label: value.trackable.tracker.label,
              include: value.trackable.tracker.include,
              uom: value.trackable.tracker.uom,
              min: value.trackable.tracker.min,
              max: value.trackable.tracker.max,
              defaultvalue: value.trackable.tracker.default,
              note: value.trackable.tracker.note,
              picks: value.trackable.tracker.picks,
              
            }}},100)

  })
  setTimeout(()=>{
  custom = temp;
  },100+(custom.length*105))
}

async function onLaunch_updateBlackList(){
  let temp = blacklist;
  temp.forEach(async(trackable,index)=>{
    let prefix = ""
    if (!trackable.tag.includes("#") && !trackable.tag.includes("@")) {
    prefix = "#";
    if (trackable.type == "person"){
      prefix = "@"
    } }
    let value = await plugin.getTrackable(prefix+trackable.tag);
    
    setTimeout(()=>{  if (trackable.type == "person"){ temp[index] = {
              id:nid(),
              username: value.id,
              name: value.trackable.person.displayName,
              emoji:"😎" ,
              displayName: value.trackable.person.displayName,
              tag: value.trackable.person.username ,
              type: "person",
              color: "",
              label: value.trackable.person.displayName,
              include: "",
              uom: "",
              min: "",
              max: "",
              defaultvalue: "",
              note: "",
              picks: "",
              
            }}
          else {temp[index] = {
              id:nid(),
              name:value.trackable.tracker.label, 
              username: value.id,
              emoji:value.trackable.tracker.emoji ,
              displayName: value.trackable.tracker.label,
              tag: value.trackable.tracker.tag ,
              type: value.trackable.tracker.type,
              color: value.trackable.tracker.color,
              label: value.trackable.tracker.label,
              include: value.trackable.tracker.include,
              uom: value.trackable.tracker.uom,
              min: value.trackable.tracker.min,
              max: value.trackable.tracker.max,
              defaultvalue: value.trackable.tracker.default,
              note: value.trackable.tracker.note,
              picks: value.trackable.tracker.picks,
              
            }}},100)

  })
  setTimeout(()=>{
  blacklist = temp;
  },100+(blacklist.length*105))
}

  async function onLaunch_changeDiscoverySelection(){
    
    if (selection =="last10d"){
      await onLaunch_getLastUsedTrackables(10)
    }
    else if (selection =="last20d"){
      await onLaunch_getLastUsedTrackables(20)
    }
    else if (selection == "last30d") {
      await onLaunch_getLastUsedTrackables(30)
    }
    else {
      PreLoadedList = [];
    }
  }

  async function onLaunch_getLastUsedTrackables(days=30){
  let hasChanges = false;
      // Search all notes
      const everyNote = await plugin.searchNotes('', new Date(), days);
      // Filter out only notes with elements that have a person
      const allNotes = everyNote.filter(note => {
        return note.elements.find(e => e.type == 'person' || e.type == 'tracker')
      })
      // Create a standalone object
      const allTrackables = { ...{} };
      // Loop over the people notes
      allNotes.forEach((note) => {
        // Loop over the elements of the notes
        // filter out on people
        note.elements.filter(e => e.type == 'person' || e.type == 'tracker').forEach(async (ele) => {
          
          
          if (!allTrackables.hasOwnProperty(ele.id)) {
            let emoji = "?"
            let value;
            if (ele.type == 'tracker'){
              value = await plugin.getTrackable("#"+ele.id);
              emoji = value.trackable.tracker.emoji
              setTimeout(()=>{  allTrackables[ele.id] = {
              username: ele.id,
              emoji:emoji,
              displayName: value.trackable.tracker.label,
              tag: value.trackable.tracker.tag ,
              type: value.trackable.tracker.type,
              color: value.trackable.tracker.color,
              label: value.trackable.tracker.label,
              include: value.trackable.tracker.include,
              uom: value.trackable.tracker.uom,
              min: value.trackable.tracker.min,
              max: value.trackable.tracker.max,
              defaultvalue: value.trackable.tracker.default,
              note: value.trackable.tracker.note,
              picks: value.trackable.tracker.picks,
              latest: note.end
            }
            hasChanges = true;},2)
            }
            else if (ele.type == 'person'){
              value = await plugin.getTrackable("@"+ele.id);
              emoji = "😎"
              setTimeout(()=>{  allTrackables[ele.id] = {
              username: ele.id,
              emoji:emoji,
              displayName: value.trackable.person.displayName,
              tag: ele.id,
              type: "person",
              color: "",
              label: value.trackable.person.displayName,
              include: "",
              uom: "",
              min: "",
              max: "",
              defaultvalue: "",
              note: "",
              picks: "",
              latest: note.end
            }
            hasChanges = true;},1)
            }

            
          } else {
            // Get this note date
            const noteDate = new Date(note.end);
            // Get last note date
            const lastUserDate = allTrackables[ele.id].latest || '1960-01-01';
            const latest = new Date(lastUserDate);

            // Is latest less than this note date?
            if (latest < noteDate) {
              hasChanges = true;
              // yes, update the latest to this notes date
              allTrackables[ele.id].latest = note.end;
            }
          }
        })
      })

      setTimeout(()=>{
      PreLoadedList = [];
      
      Object.entries(allTrackables).forEach(([key, value]) => {
      PreLoadedList.push({id:nid(),name:`${value.displayName}`,emoji:value.emoji,color:value.color,defaultvalue:value.defaultvalue,include:value.include,label:value.label,min:value.min,max:value.max,tag:value.tag,type:value.type,uom:value.uom,note:value.note,picks:value.picks})
    });
    
},200)
}



async function onLaunch_SaveSyncSelection(){
  var CombinedList = [];
  CombinedList.length = 0;
  custom.forEach((element) => {
    if (!CombinedList.some(e => e.tag === element.tag)) {
    CombinedList.push(element)}})
    PreLoadedList.forEach((element) => {
      if (!CombinedList.some(e => e.tag === element.tag)) {
    CombinedList.push(element)}})


    blacklist.forEach((element) => {
      let objIndex = CombinedList.findIndex((obj => obj.tag == element.tag));
      if (objIndex >= 0) {
        CombinedList[objIndex].type = CombinedList[objIndex].type+"_blacklisted" }
   })
   var data = {detail:[selection,custom,blacklist,CombinedList]}

   
   plugin.alert("Apple Watch Synced");
   await syncTrackables(data)
}




// ALL ON LAUNCH CODE END

function onLoaded() {
  if (!plugin) {
  plugin = new NomiePlugin({
        name: pluginname,
        emoji: pluginemoji,
        description: "Apple Watch Hub Plugin to sync data to Apple Watch App",
        uses: ['selectTrackables', 'searchNotes', 'onLaunch'],
        version: "0.5",
        addToCaptureMenu: true,
        addToMoreMenu: true,
        addToWidgets: false,
      })
  }
}




</script>

<LibLoader url={PlugiAapiUrl}
on:loaded="{onLoaded}" />
{#if mode == "modal"  || mode =="widget"}
<Theme bind:theme />
{#if inNomie}
{#if mode == "modal"}
<Header company={parent} platformName={pluginname} on:click={showMain}>
  <svelte:fragment slot="skip-to-content">
    <SkipToContent />
  </svelte:fragment>
  <HeaderUtilities>
    <HeaderGlobalAction aria-label="Settings" icon={SettingsAdjust} on:click={showSettings}/>
    <HeaderGlobalAction aria-label="Theme" icon={Sun} on:click={toggleTheme}/>
    <HeaderGlobalAction aria-label="Theme" icon={Information} on:click={showInformation}/>
  </HeaderUtilities>
</Header>

{#if view == "main"}
<Main pluginname={pluginname} pluginemoji={pluginemoji} {plugin} bind:Selection={selection} bind:CustomList={custom} bind:BlackList={blacklist} on:savetrackables={saveTrackables} on:savesynctrackables={syncTrackables}/>
{:else if view == "info"}
<Info parent={parent} pluginname={pluginname} pluginemoji={pluginemoji} on:exitinfo={showMain}/>
{:else if view == "settings"}
<Settings pluginname={pluginname} pluginemoji={pluginemoji} parent={parent} bind:synconstart={synconstart} bind:awapikey={awapikey} bind:apiaddress={apiaddress} bind:apikey={apikey} on:exitsettings={exitSettings} on:savesettings={saveSettings}/>
{/if}
{:else if mode == "widget"}
<p>Widget Placeholder</p>
{/if}
{/if}

{:else if !inNomie}
        <h1 style="text-align:center">{pluginemoji}</h1>
        <h2 style="text-align:center">{pluginname}</h2>
        <h5 style="text-align:center">This is a plugin for {parent}</h5>
        <hr>
{/if}
{#if loading}
<div class="startup">
<p>Loading....</p>
</div>
{/if}
