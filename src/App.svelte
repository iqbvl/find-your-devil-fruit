<script>
    import { onMount } from 'svelte';
    
    let devilFruit = null;
    let loading = false;
    let error = null;
    let savedFruits = [];
    
    // Load saved fruits from cookies on component mount
    onMount(() => {
        const saved = getCookie('savedDevilFruits');
        if (saved) {
            savedFruits = JSON.parse(saved);
        }
    });
    
    // Function to get cookie value
    function getCookie(name) {
        const value = `; ${document.cookie}`;
        const parts = value.split(`; ${name}=`);
        if (parts.length === 2) return parts.pop().split(';').shift();
    }
    
    // Function to set cookie
    function setCookie(name, value, days = 365) {
        const date = new Date();
        date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
        document.cookie = `${name}=${value};expires=${date.toUTCString()};path=/`;
    }
    
    // Function to fetch a random devil fruit
    async function findDevilFruit() {
        loading = true;
        error = null;
        
        try {
            const response = await fetch('https://api.api-onepiece.com/v2/fruits/en');
            if (!response.ok) throw new Error('Failed to fetch devil fruits');
            
            const data = await response.json();
            const fruits = data.fruits || [];
            
            if (fruits.length === 0) throw new Error('No devil fruits found');
            
            // Get a random fruit
            const randomIndex = Math.floor(Math.random() * fruits.length);
            devilFruit = fruits[randomIndex];
            
            // Save to saved fruits if not already there
            if (!savedFruits.some(f => f.id === devilFruit.id)) {
                savedFruits = [...savedFruits, devilFruit];
                setCookie('savedDevilFruits', JSON.stringify(savedFruits));
            }
        } catch (err) {
            error = err.message;
            console.error(err);
        } finally {
            loading = false;
        }
    }
    
    // Function to clear saved fruits
    function clearSavedFruits() {
        savedFruits = [];
        setCookie('savedDevilFruits', '');
    }
</script>

<main>
    <h1>Find Your Devil Fruit</h1>
    
    <div class="container">
        <button on:click={findDevilFruit} disabled={loading}>
            {loading ? 'Searching...' : 'Find My Devil Fruit!'}
        </button>
        
        {#if error}
            <p class="error">{error}</p>
        {/if}
        
        {#if devilFruit}
            <div class="result">
                <h2>Your Devil Fruit is: {devilFruit.name}</h2>
                <p><strong>Type:</strong> {devilFruit.type}</p>
                <p><strong>Ability:</strong> {devilFruit.ability}</p>
                <p><strong>Description:</strong> {devilFruit.description}</p>
            </div>
        {/if}
        
        {#if savedFruits.length > 0}
            <div class="saved-fruits">
                <h3>Previously Found Fruits:</h3>
                <ul>
                    {#each savedFruits as fruit}
                        <li>{fruit.name} ({fruit.type})</li>
                    {/each}
                </ul>
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