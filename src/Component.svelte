<script>
  import { getContext } from "svelte"

  export let structureSource
  export let staticStructure
  export let dataProvider
  export let sectionColumnKey
  export let sectionColumnValue
  export let itemColumn
  export let onItemClick

  const { styleable, builderStore } = getContext("sdk")
  const component = getContext("component")
  const loading = getContext("loading")

  let _selectedItemKey
  let _structure
  let _error

  const sampleJSON = {
    "sections": [
      {
        "sectionKey": 1,
        "sectionValue": "Section 1",
        "items": [
          { "itemKey": 1, "itemValue": "Item 1" },
          { "itemKey": 2, "itemValue": "Item 2" },
          { "itemKey": 3, "itemValue": "Item 3" },
          { "itemKey": 4, "itemValue": "Item 4" }
        ]
      },
      {
        "sectionKey": 2,
        "sectionValue": "Section 2",
        "items": [
          { "itemKey": 21, "itemValue": "Item 2.1" },
          { "itemKey": 22, "itemValue": "Item 2.2" },
          { "itemKey": 23, "itemValue": "Item 2.3" }
        ]
      },
      {
        "sectionKey": 3,
        "sectionValue": "Section 3",
        "items": [
          { "itemKey": 31, "itemValue": "Item 3.1" },
          { "itemKey": 32, "itemValue": "Item 3.2" },
          { "itemKey": 33, "itemValue": "Item 3.3" }
        ]
      }
    ]
  }

  $: staticStructure, fallbackJSON()

  function fallbackJSON ( ) {
    if (!staticStructure){
      builderStore.actions.updateProp("staticStructure", JSON.stringify(sampleJSON)) 
    }
  } 

  function populateStructure ( )
  {
    _structure = { "sections": [] }

    if ( structureSource === "dataProvider" && !$loading && dataProvider )
    {
      dataProvider.rows.forEach(element => {
        let _section = { "sectionKey": 0, "sectionValue": "", "items": [] }
        _section.sectionKey = element[sectionColumnKey];
        _section.sectionValue = element[sectionColumnValue];
        
        // Iterate Ralationship Item
        if ( Array.isArray(element[itemColumn]))
          element[itemColumn].forEach(itemElement => {
            _section.items.push ( {"itemKey": itemElement._id, "itemValue": itemElement.primaryDisplay  } );
          });

        _structure.sections.push(_section);
      })

    } else {
      _error = undefined
      try {
        _structure = JSON.parse(staticStructure)
      } catch ( error ) 
      {
        _error = " Error Parsing JSON "
      }
    }
  }

  function handleClick (key, value) {
    _selectedItemKey = key
    if ( onItemClick )
      onItemClick( { itemKey: key, itemValue: value } )
  }

  $: rows = $loading
    ? new Array(dataProvider?.limit > 20 ? 20 : dataProvider?.limit).fill({})
    : dataProvider?.rows

  $: populateStructure(rows, itemColumn, structureSource, staticStructure )
</script>

<div use:styleable={$component.styles}>

  {#if _error }
    <p class="error"> Error Parsing JSON Definition. You can set the property to empty to get a sample JSON Definition </p>
  {:else}
    <nav>
      <ul class="spectrum-SideNav">
        {#each _structure.sections as _section}
          <li class="spectrum-SideNav-item">
            <h2 class="spectrum-SideNav-heading">{_section.sectionValue}</h2>
            {#if _section.items.length > 0}
            <ul class="spectrum-SideNav" aria-labelledby="nav-heading-category-1">
              {#each _section.items as _item }
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <li 
                  class:is-selected={_item.itemKey === _selectedItemKey} 
                  class="spectrum-SideNav-item"
                  on:click={() => handleClick(_item.itemKey, _item.itemValue)}
                  >
                    <span class="spectrum-SideNav-itemLink"> {_item.itemValue} </span>
                </li>
              {/each}
            </ul>
            {/if}
          </li>
        {/each}
      </ul>
    </nav>
  {/if} 

</div>