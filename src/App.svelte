<script>
    import { onMount } from 'svelte';
    
    // State management
    let devilFruit = null;
    let loading = false;
    let error = null;
    let savedFruits = [];
    let apiAvailable = true;
    
    // API configuration
    const API_URL = 'https://api.api-onepiece.com/v2/fruits/en';
    const API_SOURCE = {
        name: "API One Piece",
        url: "https://api.api-onepiece.com",
        github: "https://github.com/peperunas/api-onepiece"
    };
    
    // Fallback data
    const FALLBACK_FRUITS = [
        {
            id: 'fallback-1',
            name: 'Gomu Gomu no Mi',
            type: 'Paramecia',
            ability: 'Rubber Human',
            description: 'Turns the user into a Rubber Human',
            picture: 'https://static.wikia.nocookie.net/onepiece/images/6/6a/Gomu_Gomu_no_Mi.png'
        },
        {
            id: 'fallback-2',
            name: 'Mera Mera no Mi',
            type: 'Logia',
            ability: 'Flame Human',
            description: 'Allows the user to create, control, and transform into fire',
            picture: 'https://static.wikia.nocookie.net/onepiece/images/7/7d/Mera_Mera_no_Mi.png'
        }
    ];

    // Initialize component
    onMount(() => {
        loadSavedFruits();
        checkApiStatus();
    });

    // Load saved fruits from cookies
    function loadSavedFruits() {
        const saved = getCookie('savedDevilFruits');
        if (saved) {
            try {
                savedFruits = JSON.parse(saved);
            } catch (e) {
                console.error('Failed to parse saved fruits:', e);
            }
        }
    }

    // Check API status
    async function checkApiStatus() {
        try {
            const response = await fetch(API_URL, { method: 'HEAD' });
            apiAvailable = response.ok;
        } catch {
            apiAvailable = false;
        }
    }

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
        document.cookie = `${name}=${value};expires=${date.toUTCString()};path=/;SameSite=Lax`;
    }

    // Fetch random devil fruit
    async function findDevilFruit() {
        loading = true;
        error = null;
        
        try {
            // Try API first if available
            if (apiAvailable) {
                const response = await fetch(API_URL);
                if (!response.ok) throw new Error('API request failed');
                
                const data = await response.json();
                const fruits = Array.isArray(data) ? data : (data.fruits || []);
                
                if (fruits.length > 0) {
                    devilFruit = fruits[Math.floor(Math.random() * fruits.length)];
                    saveFruit(devilFruit);
                    return;
                }
            }
            
            // Fallback to local data if API fails
            throw new Error('Using fallback data');
            
        } catch (err) {
            console.error(err);
            error = "API unavailable. Showing sample fruit.";
            devilFruit = FALLBACK_FRUITS[Math.floor(Math.random() * FALLBACK_FRUITS.length)];
            saveFruit(devilFruit);
        } finally {
            loading = false;
        }
    }

    // Save fruit to history
    function saveFruit(fruit) {
        if (!savedFruits.some(f => f.id === fruit.id)) {
            savedFruits = [...savedFruits, fruit];
            setCookie('savedDevilFruits', JSON.stringify(savedFruits));
        }
    }

    // Clear saved fruits
    function clearSavedFruits() {
        savedFruits = [];
        setCookie('savedDevilFruits', '');
    }
</script>

<main>
    <h1>Find Your One Piece Devil Fruit</h1>
    
    <div class="api-credit">
        <p>Data provided by: 
            <a href={API_SOURCE.url} target="_blank" rel="noopener noreferrer">{API_SOURCE.name}</a>
            (<a href={API_SOURCE.github} target="_blank" rel="noopener noreferrer">GitHub</a>)
        </p>
        {#if !apiAvailable}
            <p class="api-warning">⚠️ Currently using fallback data</p>
        {/if}
    </div>
    
    <div class="container">
        <button on:click={findDevilFruit} disabled={loading}>
            {#if loading}
                <span class="spinner">⌛</span> Searching...
            {:else}
                🍎 Find My Devil Fruit!
            {/if}
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
                            loading="lazy"
                            on:error={(e) => e.target.style.display = 'none'}
                        />
                    </div>
                {/if}
                
                <div class="fruit-details">
                    <p><strong>Type:</strong> <span class="type-{devilFruit.type.toLowerCase()}">{devilFruit.type}</span></p>
                    <p><strong>Ability:</strong> {devilFruit.ability}</p>
                    <p><strong>Description:</strong> {devilFruit.description}</p>
                </div>
            </div>
        {/if}
        
        {#if savedFruits.length > 0}
            <div class="saved-fruits">
                <h3>Your Devil Fruit History ({savedFruits.length})</h3>
                <div class="saved-fruits-grid">
                    {#each savedFruits as fruit (fruit.id)}
                        <div class="saved-fruit" on:click={() => devilFruit = fruit}>
                            {#if fruit.picture}
                                <img 
                                    src={fruit.picture} 
                                    alt={fruit.name} 
                                    class="saved-fruit-image"
                                    loading="lazy"
                                    on:error={(e) => e.target.style.display = 'none'}
                                />
                            {:else}
                                <div class="fruit-placeholder">{fruit.name.charAt(0)}</div>
                            {/if}
                            <div class="saved-fruit-info">
                                <strong>{fruit.name}</strong>
                                <span class="type-{fruit.type.toLowerCase()}">{fruit.type}</span>
                            </div>
                        </div>
                    {/each}
                </div>
                <button on:click={clearSavedFruits} class="clear-btn">🧹 Clear History</button>
            </div>
        {/if}
    </div>
</main>

<style>
    :global(body) {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        margin: 0;
        padding: 0;
        background-color: #f5f5f5;
        color: #333;
    }

    main {
        max-width: 800px;
        margin: 0 auto;
        padding: 2rem;
        background-color: white;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        border-radius: 8px;
        margin-top: 2rem;
        margin-bottom: 2rem;
    }

    h1 {
        color: #e63946;
        text-align: center;
        margin-bottom: 1.5rem;
        font-size: 2.5rem;
    }

    .api-credit {
        text-align: center;
        margin-bottom: 2rem;
        font-size: 0.9rem;
        color: #666;
    }

    .api-credit a {
        color: #e63946;
        text-decoration: none;
        font-weight: bold;
    }

    .api-credit a:hover {
        text-decoration: underline;
    }

    .api-warning {
        color: #ff9800;
        margin-top: 0.5rem;
        font-weight: bold;
    }

    button {
        background-color: #e63946;
        color: white;
        border: none;
        padding: 0.75rem 1.5rem;
        font-size: 1rem;
        border-radius: 4px;
        cursor: pointer;
        margin: 0.5rem;
        transition: all 0.3s ease;
        display: inline-flex;
        align-items: center;
        gap: 0.5rem;
    }

    button:hover {
        background-color: #c1121f;
        transform: translateY(-2px);
    }

    button:disabled {
        background-color: #cccccc;
        cursor: not-allowed;
        transform: none;
    }

    .spinner {
        animation: spin 1s linear infinite;
    }

    @keyframes spin {
        from { transform: rotate(0deg); }
        to { transform: rotate(360deg); }
    }

    .error {
        color: #e63946;
        font-weight: bold;
        text-align: center;
        padding: 1rem;
        background-color: #ffebee;
        border-radius: 4px;
        margin: 1rem 0;
    }

    .result {
        background-color: #f8f9fa;
        border-radius: 8px;
        padding: 1.5rem;
        margin: 1.5rem 0;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    }

    .result h2 {
        color: #e63946;
        margin-top: 0;
        text-align: center;
    }

    .fruit-image-container {
        text-align: center;
        margin: 1rem 0;
    }

    .fruit-image {
        max-width: 200px;
        max-height: 200px;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        object-fit: contain;
        background-color: #f0f0f0;
    }

    .fruit-details {
        margin-top: 1.5rem;
    }

    .fruit-details p {
        margin: 0.5rem 0;
        line-height: 1.6;
    }

    .type-paramecia {
        color: #4caf50;
        font-weight: bold;
    }

    .type-logia {
        color: #2196f3;
        font-weight: bold;
    }

    .type-zoan {
        color: #ff9800;
        font-weight: bold;
    }

    .saved-fruits {
        margin-top: 2rem;
        border-top: 1px solid #eee;
        padding-top: 1.5rem;
    }

    .saved-fruits h3 {
        text-align: center;
        color: #666;
        margin-bottom: 1rem;
    }

    .saved-fruits-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
        gap: 1rem;
        margin: 1rem 0;
    }

    .saved-fruit {
        background-color: #f8f9fa;
        border-radius: 8px;
        padding: 0.75rem;
        text-align: center;
        transition: all 0.3s ease;
        cursor: pointer;
    }

    .saved-fruit:hover {
        transform: translateY(-3px);
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    .saved-fruit-image {
        width: 80px;
        height: 80px;
        object-fit: contain;
        border-radius: 4px;
        margin-bottom: 0.5rem;
        background-color: #f0f0f0;
    }

    .fruit-placeholder {
        width: 80px;
        height: 80px;
        margin: 0 auto 0.5rem;
        background-color: #e63946;
        color: white;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 2rem;
        font-weight: bold;
    }

    .saved-fruit-info {
        font-size: 0.9rem;
    }

    .saved-fruit-info strong {
        display: block;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }

    .clear-btn {
        background-color: #666;
        margin-top: 1rem;
        display: block;
        margin-left: auto;
        margin-right: auto;
    }

    .clear-btn:hover {
        background-color: #444;
    }

    @media (max-width: 600px) {
        main {
            padding: 1rem;
            margin: 1rem;
        }
        
        h1 {
            font-size: 2rem;
        }
        
        .saved-fruits-grid {
            grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
        }
    }
</style>