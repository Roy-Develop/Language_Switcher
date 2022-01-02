<script>
    //import { REPL_MODE_SLOPPY } from 'node:repl';
    
    import {onMount} from 'svelte';
    import { each } from 'svelte/internal';
    
    import * as lanJson from './languages.json';
    
    let scoops = 'radio_languages'; 

    let languages = lanJson['default'];
    
    let storedData = JSON.parse(localStorage.getItem("languageSwitcher"));
    let choice = 'en-uk';
    
    if(storedData == null){
        storedData = {};
        resetPreferences();
    }
    else{
        if(Object.keys(storedData).length != languages.length){
            languages.forEach(element => {
                if(storedData[element] == undefined) 
                    storedData[element] = 'N10000001';
            });
        }
            
    }

    let url = window.location.href; //get current URL
    let ext_id = chrome.runtime.id;
    preload();
    changeLanguage();

    function onLanChoice(lan) { //change language as user choose it
        storedData[lan] = storedData[lan].charAt(0) + String(Number(storedData[lan].slice(1)) + 1);
        storedData['lastChoice'] = lan;
        localStorage.setItem("languageSwitcher", JSON.stringify(storedData));
        changeLanguage(lan);
    }

    function changeLanguage(lan){ //change searching language
        let currentLanguage;
        if(lan === undefined){ //if it is the first search, if favourite is setted use it, otherwise no change
            if(url.match(/hl=(.+)/) === null && url.match(/gl=(.+)/) === null) //if no language is in query
                currentLanguage = languages[0];
            else
                return;
        }
        else{
            currentLanguage = lan;
        }

        
        let oldLanguageH = url.match(/hl=(.+)/);
        let oldLanguageG = url.match(/gl=(.+)/);
        let newUrl = url;

        if(oldLanguageH === null && url.match(/search\?/) != null){//host language
            newUrl = newUrl+ "&hl=" + currentLanguage.split('-')[0];
        }  
        else
            newUrl = newUrl.replace(/hl=(..)/,"hl=" + currentLanguage.split('-')[0]);
        
        if(oldLanguageG === null && url.match(/search\?/) != null){ //geo language
            newUrl = newUrl+ "&gl=" + currentLanguage.split('-')[1];
        }
        else
            newUrl = newUrl.replace(/gl=(..)/, "gl=" + currentLanguage.split('-')[1]);

        if(newUrl != url)
            window.location.href = newUrl;
    }

    function changeFavourite(lan){ //setting favourite and default language and changing icon
        if(storedData[lan].charAt(0) == 'N'){
            storedData[lan] = storedData[lan].replace('N', 'Z')
        }
        else
            storedData[lan] = storedData[lan].replace('Z', 'N')

        if(document.getElementById(lan).className.includes('empty'))
            document.getElementById(lan).className = 'icon-star';
        else
            document.getElementById(lan).className = 'icon-star-empty';

        localStorage.setItem("languageSwitcher", JSON.stringify(storedData));
    }


    
    function resetPreferences(){ //reset all preferences
        languages.forEach(element => {
                    storedData[element] = 'N10000001';
        });
        storedData['lastChoice'] = null;
        localStorage.setItem("languageSwitcher", JSON.stringify(storedData));
    }


    function preload() { // set data before displaying     
        languages.sort((a, b) => JSON.parse(localStorage.getItem('languageSwitcher'))[b].localeCompare(JSON.parse(localStorage.getItem('languageSwitcher'))[a]));
    }

    function defaultImage(el){
        el[0].src = 'chrome-extension://'+{ext_id}+'/images/default.png';
    }
    
    function toggleDrop(){
        let el = document.getElementById('dropdownMenu').style;
        if(el.display == 'none')
            el.display = 'block';
        else
            el.display = 'none';
    }
    
    onMount(() => {

        let father = document.getElementById('before-appbar');
        const son = document.getElementById('LanguageSwitcher');
        if(father == null){
            father = document.getElementsByClassName('ndYZfc')[0];
            if(father == null)
                father = document.getElementsByTagName('body');
            father.prepend(son);
        }
        else
            father.appendChild(son);
        
    });

    

</script>
    
<link href="//netdna.bootstrapcdn.com/font-awesome/3.2.1/css/font-awesome.css" rel="stylesheet"> 

<div id='LanguageSwitcher' style="margin-left: 169px;">
    <h4>
        Language: 
    </h4>
    <table style="border: 1px solid rgb(26, 115, 232);    border-radius: 25px;    padding: 5px;"> 
        <tr>
            <td style="position: relative;top: -1px;">
            {#each languages as lan, i} 
                {#if i < 6}
                    <label>
                        <input on:click={() => onLanChoice(lan)} type=radio bind:group={storedData['lastChoice']} value={lan} style="position: relative;top: 2px;">&nbsp;<img src='chrome-extension://{ext_id}/images/{lan.split('-')[1]}.png' alt='{lan} flag' onerror="this.src='chrome-extension://{ext_id}/images/default.png';" style="position:relative;top: 5px;">
                    </label>&nbsp;
                    <button on:click={() => changeFavourite(lan)}>
                        {#if storedData[lan][0]=='N'}
                            <i id={lan} class="icon-star-empty"></i>
                        {:else}
                            {#if storedData[lan][0]=='Z'}
                                <i id={lan} class="icon-star"></i>
                            {/if}
                        {/if}
                    </button>&nbsp;{lan.split('-')[1]}
                &nbsp;
                {/if}
                
                
            {/each}
            
            {#if languages.length > 6}

                <div class="dropdown">
                    <button on:click={() => toggleDrop()} type="button" style="background: white; min-width: 64px; border: 1px solid rgb(26, 115, 232); padding-bottom: 3px; border-radius: 25px;    padding: 5px;">
                        <i class="icon-circle-arrow-down">&nbsp; Other &nbsp;</i>
                    </button>
                    <div id='dropdownMenu'  style="display: None; position: absolute; z-index: 1;">
                        {#each languages as lan, i} 
                            {#if i>5}
                                <button value={lan} on:click={() => onLanChoice(lan)} type="button" style="background: white; min-width: 64px; border: 1px solid rgb(26, 115, 232); padding-bottom: 7px;"> 
                                    <img src='chrome-extension://{ext_id}/images/{lan.split('-')[1]}.png' onerror="this.onerror=null;this.src='chrome-extension://{ext_id}/images/default.png';" alt='{lan} flag'  style="position:relative;top: 5px;">  {lan.split('-')[1]}
                                </button>
                                <br>
                            {/if}             
                        {/each}   
                    </div>
                </div>


            {:else if languages.length == 6}
                <label>
                    <input on:click={() => onLanChoice(languages[6])} type=radio bind:group={storedData['lastChoice']} value={languages[6]} style="position: relative;top: 2px;">&nbsp;<img src='chrome-extension://{ext_id}/images/{languages[6].split('-')[1]}.png' alt='{languages[6]} flag' onerror="this.src='chrome-extension://{ext_id}/images/default.png';" style="position:relative;top: 5px;">
                </label>&nbsp;
                <button on:click={onLanChoice([this, languages[6]])}>
                    {#if storedData[languages[6]][0]=='N'}
                        <i class="icon-star-empty"></i>
                    {:else}
                        {#if storedData[languages[6]][0]=='Z'}
                            <i class="icon-star"></i>
                        {/if}
                    {/if}
                </button>&nbsp;{languages[6].split('-')[1]}

            {/if}
            </td>    
            <td>
                <button on:click={() => resetPreferences() } style="background: white; min-width: 64px; border: 1px solid rgb(26, 115, 232); padding-bottom: 3px; border-radius: 25px;    padding: 5px;">Reset</button>

            </td>

        </tr>
    </table>    
    
</div>
    