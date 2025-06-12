<script>
    import { onMount } from 'svelte';
    
    let devilFruit = null;
    let loading = false;
    let error = null;
    let savedFruits = [];
    
    // API Source Attribution
    const API_SOURCE = {
        name: "API One Piece",
        url: "https://api.api-onepiece.com",
        github: "https://github.com/peperunas/api-onepiece"
    };
    
    // Load saved fruits from cookies
    onMount(() => {
        const saved = getCookie('savedDevilFruits');
        if (saved) {
            savedFruits = JSON.parse(saved);
        }
    });
    
    // Get cookie value
    function getCookie(name) {
        const value = `; ${document.cookie}`;
        const parts = value.split(`; ${name}=`);
        if (parts.length === 2) return parts.pop().split(';').shift();
    }
    
    // Set cookie
    function setCookie(name, value, days = 365) {
        const date = new Date();
        date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
        document.cookie = `${name}=${value};expires=${date.toUTCString()};path=/`;
    }
    
    // Fetch a random Devil Fruit
    async function findDevilFruit() {
        loading = true;
        error = null;
        
        try {
            const response = await fetch('https://api.api-onepiece.com/v2/fruits/en');
            if (!response.ok) throw new Error('API request failed');
            
            const data = await response.json();
            const fruits = Array.isArray(data) ? data : (data.fruits || []);
            
            if (fruits.length === 0) throw new Error('No fruits available');
            
            const randomIndex = Math.floor(Math.random() * fruits.length);
            devilFruit = fruits[randomIndex];
            
            if (!savedFruits.some(f => f.id === devilFruit.id)) {
                savedFruits = [...savedFruits, devilFruit];
                setCookie('savedDevilFruits', JSON.stringify(savedFruits));
            }
        } catch (err) {
            error = "Failed to fetch Devil Fruits. Showing a default fruit.";
            console.error(err);
            
            // Fallback data (Gomu Gomu no Mi)
            devilFruit = {
                id: 'fallback-1',
                name: 'Gomu Gomu no Mi',
                type: 'Paramecia',
                ability: 'Rubber Human',
                description: 'Turns the user into a Rubber Human',
                picture: 'https://static.wikia.nocookie.net/onepiece/images/6/6a/Gomu_Gomu_no_Mi.png'
            };
        } finally {
            loading = false;
        }
    }
    
    // Clear saved fruits
    function clearSavedFruits() {
        savedFruits = [];
        setCookie('savedDevilFruits', '');
    }
</script>

<main>
    <h1>Find Your Devil Fruit</h1>
    
    <div class="api-credit">
        <p>Data provided by: 
            <a href={API_SOURCE.url} target="_blank" rel="noopener noreferrer">{API_SOURCE.name}</a>
            (<a href={API_SOURCE.github} target="_blank" rel="noopener noreferrer">GitHub</a>)
        </p>
    </div>
    
    <div class="container">
        <button on:click={findDevilFruit} disabled={loading}>
            {loading ? 'Searching...' : 'Find My Devil Fruit!'}
        </button>
        
        {#if error}
            <p class="error">{error}</p>
        {/if}
        
        {#if devilFruit}
            <div class="result">
                <h2>{devilFruit.name}</h2>
                
                {#if devilFruit.picture}
                    <div class="fruit-image-container">
                        <img 
                            src={devilFruit.picture} 
                            alt={devilFruit.name} 
                            class="fruit-image"
                            on:error={(e) => e.target.style.display = 'none'}
                        />
                    </div>
                {/if}
                
                <div class="fruit-details">
                    <p><strong>Type:</strong> {devilFruit.type}</p>
                    <p><strong>Ability:</strong> {devilFruit.ability}</p>
                    <p><strong>Description:</strong> {devilFruit.description}</p>
                </div>
            </div>
        {/if}
        
        {#if savedFruits.length > 0}
            <div class="saved-fruits">
                <h3>Previously Found:</h3>
                <div class="saved-fruits-grid">
                    {#each savedFruits as fruit}
                        <div class="saved-fruit">
                            {#if fruit.picture}
                                <img 
                                    src={fruit.picture} 
                                    alt={fruit.name} 
                                    class="saved-fruit-image"
                                    on:error={(e) => e.target.style.display = 'none'}
                                />
                            {/if}
                            <div class="saved-fruit-info">
                                <strong>{fruit.name}</strong>
                                <span>({fruit.type})</span>
                            </div>
                        </div>
                    {/each}
                </div>
                <button on:click={clearSavedFruits}>Clear History</button>
            </div>
        {/if}
    </div>
</main>

<style>
    main {
        text-align: center;
        padding: 1em;
        max-width: 800px;
        margin: 0 auto;
    }
    
    h1 {
        color: #e63946;
        margin-bottom: 2rem;
    }
    
    button {
        background-color: #e63946;
        color: white;
        border: none;
        padding: 0.5rem 1rem;
        font-size: 1rem;
        border-radius: 4px;
        cursor: pointer;
        margin: 0.5rem;
    }
    
    button:hover {
        background-color: #c1121f;
    }
    
    button:disabled {
        background-color: #cccccc;
        cursor: not-allowed;
    }
    
    .result {
        background-color: #f8f9fa;
        border-radius: 8px;
        padding: 1rem;
        margin: 1rem 0;
        text-align: left;
    }
    
    .error {
        color: #e63946;
        font-weight: bold;
    }
    
    .saved-fruits {
        margin-top: 2rem;
        border-top: 1px solid #ddd;
        padding-top: 1rem;
    }
    
    ul {
        list-style-type: none;
        padding: 0;
    }
    
    li {
        margin: 0.5rem 0;
    }
</style>