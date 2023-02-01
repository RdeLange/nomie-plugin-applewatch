<script>
    import { createEventDispatcher } from 'svelte';
    import {
      Button,
      Content,
        Grid,
        Row,
        Column,
        Tile,
        TextInput,
        Toggle
    } from "carbon-components-svelte";
   
    export let pluginname;
    export let pluginemoji;
    export let apiaddress;
    export let awapiaddress;
    export let apikey;
    export let awapikey;
    export let synconstart = false;
    export let parent;

   let open=true;
   
   const dispatch = createEventDispatcher();

   $: if(synconstart){
    console.log(synconstart);
   }

   function exitSettings(){
        dispatch("exitsettings");
        open = false;
    }

    function saveSettings(){
        dispatch("savesettings");
        dispatch("exitsettings");
        open = false;
    } 
</script>

<Content>
    <Grid>
      <Row>
        <Column>
          <h1 style="text-align:center">{pluginemoji}</h1>
          <h2 style="text-align:center">{pluginname}</h2>
          <h5 style="text-align:center">Plugin Settings</h5>
          <hr>
        </Column>
      </Row>
    </Grid>
    <Row>
      <Column>
        <Tile>
          <p>Configure {parent} API</p>
          <h5 style="font-size:0.8em; font-weight:300">This section configures the API your Apple Watch will use to log to {parent}. You can find the API address and key in the {parent} API settings section</h5>
          <br>
          <TextInput bind:value={apiaddress} size="sm" inline labelText="API address" placeholder="Leave empty for default.." />
          <TextInput bind:value={apikey} size="sm" inline labelText="API key" placeholder="Enter the {parent} API key" />
          <br>
          <p>Configure {parent} Apple Watch API</p>
          <h5 style="font-size:0.8em; font-weight:300">This section configures the API the plugin will use to conect to your Apple Watch {parent} App. You can find the API key in the {parent} Apple Watch App</h5>
          <br>
          <TextInput bind:value={awapiaddress} size="sm" inline labelText="AW API domain" placeholder="Leave empty for default.." />
          <TextInput bind:value={awapikey} size="sm" inline labelText="AW API key" placeholder="Enter AW {parent}App API key" />
          <br>
          <p>Sync at {parent} Start</p>
          <h5 style="font-size:0.8em; font-weight:300">Indicate if you would like to automatically sync your Trackable selection regularly in the background</h5>
          
          <Toggle bind:toggled={synconstart} size="sm" labelText="Sync with AW" hideLabel/>
        </Tile>
      </Column>
    </Row>
    <Row>
      <Column>
          <br>
           <span><Button kind="secondary" on:click={exitSettings} style="float: left;">Exit</Button></span>
           <br>
      </Column>
    <Column>
        <br>
         <span><Button on:click={saveSettings} style="float: right;">Save</Button></span>
         <br>
    </Column>
</Row>
  </Content>



  <style>
    h2 {
       margin: 0;
       padding: 0;
       font-size: 2.5em;
       font-weight: 400;
   }

 </style>

