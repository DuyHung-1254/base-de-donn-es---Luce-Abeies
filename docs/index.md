<!-- Tìm kiếm Tác giả, Tác phẩm và Thể loại -->
![Image](./asset/frise_luce.png)

<div style="display: flex; align-items: center;">
    <input type="text" id="search-input" placeholder="Entrez le nom de l'auteur, le titre de l'œuvre ou le genre" style="flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 5px; margin-right: 10px;">
    <select id="search-type" style="padding: 10px; border: 1px solid #ccc; border-radius: 5px; background-color: #f9f9f9;">
        <option value="author">Auteur</option>
        <option value="work">Titre</option>
        <option value="genre">Genre</option>
    </select>
</div>
<div id="search-results" style="margin-top: 20px;"></div>



<style>
    #search-results div {
        margin-bottom: 20px;
        padding: 10px;
        border: 1px solid #ddd;
        border-radius: 5px;
        background-color: #f9f9f9;
        cursor: pointer;
    }
</style>

<script src="https://cdnjs.cloudflare.com/ajax/libs/lunr.js/2.3.9/lunr.min.js"></script>

<script>
    let idx;
    let documents;

    // Tải dữ liệu JSON
    fetch('Base_Luce.json')
        .then(response => response.json())
        .then(data => {
            documents = data;
            createIndex();
        });

    function createIndex() {
        const searchType = document.getElementById('search-type').value;

        idx = lunr(function () {
            this.ref('id');
            
            if (searchType === 'author') {
                this.field('prénom auteur');
                this.field('auteurMIN');
            } else if (searchType === 'work') {
                this.field('titre');
            } else if (searchType === 'genre') {
                this.field('genre');
            }

            documents.forEach((doc, idx) => {
                this.add({ ...doc, id: idx });
            });
        });

        performSearch();
    }

function performSearch() {
    const query = document.getElementById('search-input').value.trim(); // Xóa khoảng trắng ở đầu và cuối
    const resultsDiv = document.getElementById('search-results');
    resultsDiv.innerHTML = '';

    // Kiểm tra nếu từ khóa trống thì không thực hiện tìm kiếm
    if (query === '') {
        resultsDiv.innerHTML = '<p>Veuillez entrer un terme de recherche.</p>'; // Hiển thị hướng dẫn nhập từ khóa
        return;
    }

    const results = idx.search(query);

    if (results.length > 0) {
        results.forEach(result => {
            const doc = documents[result.ref];
            const resultItem = document.createElement('div');

            resultItem.addEventListener('click', () => {
                window.location.href = `detailPage.html?id=${result.ref}`;
            });

            const auteur = `${doc["prénom auteur"] || ""} ${doc["auteurMIN"] || ""}`.replace(/\bnull\b/gi, "").trim();
            const titre = doc["titre"] || "";
            const date = doc["date"] || "";
            const editeur = doc["éditeur"] || "";
            const lieu = doc["lieu"] || "";
            const genre = doc["genre"] || "";

            resultItem.innerHTML = `
                <strong>Auteur:</strong> ${auteur} <br>
                <strong>Titre:</strong> ${titre} <br>
                <strong>Édition:</strong> ${editeur} <br>
                <strong>Lieu:</strong> ${lieu} <br>
                <strong>Année:</strong> ${date} <br>
                <strong>catégorie:</strong> ${genre}
            `;
            resultsDiv.appendChild(resultItem);
        });
    } else {
        resultsDiv.innerHTML = '<p>Aucun résultat trouvé.</p>';
    }
}


    document.getElementById('search-type').addEventListener('change', createIndex);
    document.getElementById('search-input').addEventListener('input', performSearch);
</script>
