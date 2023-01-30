<script>
    import { createEventDispatcher } from 'svelte';
    import {
      Button,
      Content,
        Grid,
        Row,
        Column,
        Select, 
        SelectItem,
        Tile
    } from "carbon-components-svelte";
    import Add from "carbon-icons-svelte/lib/Add.svelte";
    import Save from "carbon-icons-svelte/lib/Save.svelte";
    import Repeat from "carbon-icons-svelte/lib/Repeat.svelte";
    import SortableList from 'svelte-sortable-list';
    import SortableListItem from '../components/SortableListItem.svelte';
    import nid from "../modules/nid/nid";
   
    export let pluginname;
    export let pluginemoji;
    export let plugin;
    export let Selection = "None";
    export let CustomList = [];
    export let BlackList = [];

   let open=true;
   let PreLoadedList = []
   
   const dispatch = createEventDispatcher();


    //start AW Plugin Code
    changeDiscoverySelection(Selection);
    updateCustomList();
    updateBlackList();
    let refreshed = true;
    let refreshedbl = true;
const sortList = ev => {CustomList = ev.detail};
const sortListbl = ev => {BlackList = ev.detail};



async function getLastUsedTrackables(days=30){
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
    
      refreshed = false;
  setTimeout(()=>{refreshed = true;},1)
},200)
}

function changeDiscoverySelection(event){
  Selection = event;
  if (event =="last10d"){
    getLastUsedTrackables(10)
  }
  else if (event =="last20d"){
    getLastUsedTrackables(20)
  }
  else if (event == "last30d") {
    getLastUsedTrackables(30)
  }
  else {
    PreLoadedList = [];
  }
}

async function addTrackables(){
  let res = await plugin.selectTrackables("tracker");
  res.forEach(tracker => {
    const found = CustomList.find(element => element.tag == tracker.tracker.tag);
    if (found == undefined){
   
  CustomList.push({id:nid(),name: tracker.tracker.label,emoji:tracker.tracker.emoji,
    tag: tracker.tracker.tag ,
              type: tracker.tracker.type,
              color: tracker.tracker.color,
              label: tracker.tracker.label,
              include: tracker.tracker.include,
              uom: tracker.tracker.uom,
              min: tracker.tracker.min,
              max: tracker.tracker.max,
              defaultvalue: tracker.tracker.default,
              note: tracker.tracker.note,
              picks: tracker.tracker.picks})

            }});            
            refreshed = false;
  setTimeout(()=>{refreshed = true;},1)
          }

async function addPersons(){
  let res = await plugin.selectTrackables("person");
  res.forEach(tracker => {
    const found = CustomList.find(element => element.tag == tracker.person.username);
    if (found == undefined){
   
   CustomList.push({id:nid(),name: tracker.person.displayName,emoji:"😎",
     tag: tracker.person.username ,
               type: "person",
               color: "",
               label: tracker.person.displayName,
               include: "",
               uom: "",
               min: "",
               max: "",
               defaultvalue: "",
              note: "",
              picks: ""})
 
             }});            
             refreshed = false;
   setTimeout(()=>{refreshed = true;},1)
}

async function addTrackablesbl(){
  let res = await plugin.selectTrackables("tracker");
  res.forEach(tracker => {
    const found = BlackList.find(element => element.tag == tracker.tracker.tag);
    if (found == undefined){
   
  BlackList.push({id:nid(),name: tracker.tracker.label,emoji:tracker.tracker.emoji,
    tag: tracker.tracker.tag ,
              type: tracker.tracker.type,
              color: tracker.tracker.color,
              label: tracker.tracker.label,
              include: tracker.tracker.include,
              uom: tracker.tracker.uom,
              min: tracker.tracker.min,
              max: tracker.tracker.max,
              defaultvalue: tracker.tracker.default,
              note: tracker.tracker.note,
              picks: tracker.tracker.picks})

            }});            
            refreshedbl = false;
  setTimeout(()=>{refreshedbl = true;},1)
          }

async function addPersonsbl(){
  let res = await plugin.selectTrackables("person");
  res.forEach(tracker => {
    const found = BlackList.find(element => element.tag == tracker.person.username);
    if (found == undefined){
   
   BlackList.push({id:nid(),name: tracker.person.displayName,emoji:"😎",
     tag: tracker.person.username ,
               type: "person",
               color: "",
               label: tracker.person.displayName,
               include: "",
               uom: "",
               min: "",
               max: "",
               defaultvalue: "",
              note: "",
              picks: ""})
 
             }});            
             refreshedbl = false;
   setTimeout(()=>{refreshedbl = true;},1)
}

function SaveSelection(){
  var CombinedList = [];
  CombinedList.length = 0;
  CustomList.forEach((element) => {
    if (!CombinedList.some(e => e.tag === element.tag)) {
    CombinedList.push(element)}})
    PreLoadedList.forEach((element) => {
      if (!CombinedList.some(e => e.tag === element.tag)) {
    CombinedList.push(element)}})
   
     // adjust blacklisted
    BlackList.forEach((element) => {
      let objIndex = CombinedList.findIndex((obj => obj.tag == element.tag));
      if (objIndex >= 0) {
        CombinedList[objIndex].type = CombinedList[objIndex].type+"_blacklisted" }
   })

 dispatch("savetrackables",[Selection,CustomList, BlackList])
 changeDiscoverySelection(Selection);
    updateCustomList();
    updateBlackList();
}

function SaveSyncSelection(){
  var CombinedList = [];
  CombinedList.length = 0;
  CustomList.forEach((element) => {
    if (!CombinedList.some(e => e.tag === element.tag)) {
    CombinedList.push(element)}})
    PreLoadedList.forEach((element) => {
      if (!CombinedList.some(e => e.tag === element.tag)) {
    CombinedList.push(element)}})


    BlackList.forEach((element) => {
      let objIndex = CombinedList.findIndex((obj => obj.tag == element.tag));
      if (objIndex >= 0) {
        CombinedList[objIndex].type = CombinedList[objIndex].type+"_blacklisted" 
    }
   })
 dispatch("savesynctrackables",[Selection,CustomList,BlackList,CombinedList])
 changeDiscoverySelection(Selection);
    updateCustomList();
    updateBlackList();
}

function deleteItem(item){
let index = item.index;
CustomList.splice(index,1);
refreshed = false;
   setTimeout(()=>{refreshed = true;},1)

}

function deleteItembl(item){
let index = item.index;
BlackList.splice(index,1);
refreshedbl = false;
   setTimeout(()=>{refreshedbl = true;},1)

}

async function updateCustomList(){
  let temp = CustomList;
  temp.forEach(async(trackable,index)=>{
    let prefix = ""
    if (!trackable.tag.includes("#") && !trackable.tag.includes("@")) {
    prefix = "#";
    if (trackable.type == "person"){
      prefix = "@"
    } }
    let value = await plugin.getTrackable(prefix+trackable.tag);
    console.log("⌚️=> Trackables Update Discovery")
    
    
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
  CustomList = temp;
  refreshed = false;
   setTimeout(()=>{refreshed = true;},1)},100+(CustomList.length*105))
}

async function updateBlackList(){
  let temp = BlackList;
  temp.forEach(async(trackable,index)=>{
    let prefix = ""
    if (!trackable.tag.includes("#") && !trackable.tag.includes("@")) {
    prefix = "#";
    if (trackable.type == "person"){
      prefix = "@"
    } }
    let value = await plugin.getTrackable(prefix+trackable.tag);
    console.log("⌚️=> Trackables Update Discovery for BlackList")
    
    
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
  BlackList = temp;
  refreshedbl = false;
   setTimeout(()=>{refreshedbl = true;},1)},100+(BlackList.length*105))
}

</script>

<Content>
    <Grid>
      <Row>
        <Column>
          <h1 style="text-align:center">{pluginemoji}</h1>
          <h2 style="text-align:center">{pluginname}</h2>
          <h5 style="text-align:center">Loaded Succesfully</h5>
          <hr>
        </Column>
      </Row>
      <Row>
        <Column>
          <Select
  labelText="Discovered Trackables"
  size="sm"
  selected={Selection}
  on:change={(e) => changeDiscoverySelection(e.detail)}
>
  <SelectItem value="none" text="None" />
  <SelectItem value="last10d" text="Last 10 Days" />
  <SelectItem value="last20d" text="Last 20 Days" />
  <SelectItem value="last30d" text="Last 30 Days" />
</Select>
<Tile>
  {#if PreLoadedList.length > 0}
  {#each PreLoadedList as trackable}
  <p>{trackable.emoji} {trackable.name}</p>
  {/each}
  {:else}
  <p>∅ No Trackables</p>
  {/if}

</Tile>
        </Column>
        </Row>
        <Row>
        <Column>
          <h6 style="font-weight:300;font-size:85%;padding-bottom:8px">Custom Trackables</h6>
         
          <Button style="width:49%" icon={Add} size="small" kind="tertiary" on:click={()=>{addTrackables()}}>Trackable</Button>
         
            <Button style="width:49%" icon={Add} size="small" kind="tertiary" on:click={()=>{addPersons()}}>Person</Button>
          
        <Tile>
          {#if CustomList.length > 0}
          <SortableList 
          bind:list={CustomList} 
          key="id" 
          on:sort={sortList}
          let:item
          let:index
      >
      {#if refreshed}
      <SortableListItem {item} {index} on:delete={()=>{deleteItem({index})}}/>
          {/if}
      </SortableList>
      {:else}
      <p>∅ No Trackables</p>
      {/if}
        </Tile>
        <br>
        <h6 style="font-weight:300;font-size:85%;padding-bottom:8px">Trackables Blacklist</h6>
         
          <Button style="width:49%" icon={Add} size="small" kind="tertiary" on:click={()=>{addTrackablesbl()}}>Trackable</Button>
         
            <Button style="width:49%" icon={Add} size="small" kind="tertiary" on:click={()=>{addPersonsbl()}}>Person</Button>
          
        <Tile style="background:black;color:white">
          {#if BlackList.length > 0}
          <SortableList 
          bind:list={BlackList} 
          key="id" 
          on:sort={sortListbl}
          let:item
          let:index
      >
      {#if refreshedbl}
      <SortableListItem {item} {index} on:delete={()=>{deleteItembl({index})}}/>
          {/if}
      </SortableList>
      {:else}
      <p>∅ No Trackables</p>
      {/if}
        </Tile>
        </Column>
      </Row>
     <br>
    <Row>
      <Column>
        <Button icon={Save} style="width:100%" on:click={()=>{SaveSelection()}}>Save</Button>
      </Column>
      <Column>
        <Button icon={Repeat} style="width:100%" on:click={()=>{SaveSyncSelection()}}>Save&Sync</Button>
      </Column>
    </Row>
    </Grid>
    

  </Content>



  <style>
    h2 {
       margin: 0;
       padding: 0;
       font-size: 2.5em;
       font-weight: 400;
   }




 </style>

