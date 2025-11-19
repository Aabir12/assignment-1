global $baseUri;

$carts = $_SESSION['panier'] ?? [];

echo "<h2>Votre panier</h2>";

if (empty($carts)) {
    echo "<p>Votre panier est vide.</p>";
} else {

    echo "<table border='1' cellpadding='10'>";
    echo "<tr>
            <th>Image</th>
            <th>Nom</th>
            <th>Prix</th>
            <th>Quantité</th>
            <th>Total</th>
          </tr>";

    foreach ($carts as $item) {

        $totalLine = $item['price'] * $item['quantity'];

        echo "<tr>
                <td><img src='{$baseUri}{$item['image']}' width='80'></td>
                <td>{$item['name']}</td>
                <td>{$item['price']} €</td>
                <td>{$item['quantity']}</td>
                <td>{$totalLine} €</td>
              </tr>";
    }




    function addToCart() {
    global $pdo;
    global $baseUri;

    if(!isset($_GET['id']) || empty($_GET['id'])){
        die("Product ID missing.");
    }

    $id = intval($_GET['id']);

    // Get product from DB
    $stmt = $pdo->query("SELECT * FROM products WHERE id = $id");
    $product = $stmt->fetch(PDO::FETCH_ASSOC);

    if (!$product) {
        die("Produit introuvable !");
    }

    // Initialize cart
    if (!isset($_SESSION['panier'])) {
        $_SESSION['panier'] = [];
    }

    // If product already in cart → increase quantity
    if (isset($_SESSION['panier'][$id])) {
        $_SESSION['panier'][$id]['quantity']++;
    } else {
        // Add new product
        $_SESSION['panier'][$id] = [
            'id' => $product['id'],
            'name' => $product['name'],
            'price' => $product['price'],
            'image' => $product['image'],
            'quantity' => 1
        ];
    }

    header("Location: $baseUri/cart");
    exit;
}


function showCart() {
    global $rootUri;
    $cart = $_SESSION['panier'] ?? [];
    require_once $rootUri."\\views\\cart.php";
}


    echo "</table>";
}
?>

<a href="<?=$baseUri?>/cart?action=checkout">Valider la commande</a>
