<?php
// PDO ব্যবহার করে MySQL এর সাথে সংযোগ


<?php
require 'db.php';
if ($argc < 3) exit("Usage: php setup_user.php username password\n");

$username = $argv[1];
$password = $argv[2];
$hash = password_hash($password, PASSWORD_DEFAULT);

// User insert
$stmt = $pdo->prepare("INSERT INTO users (username,password_hash) VALUES (?,?)");
$stmt->execute([$username,$hash]);
echo "User created\n";


:

$argv[1] = username, $argv[2] = password


<?php
session_start();
require 'db.php';

if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $csrf = $_POST['csrf_token'] ?? '';
    if (!hash_equals($_SESSION['csrf_token'], $csrf)) {
        die("Invalid CSRF token");
    }

    $username = $_POST['username'] ?? '';
    $password = $_POST['password'] ?? '';

    $stmt = $pdo->prepare("SELECT id, username, password_hash FROM users WHERE username=?");
    $stmt->execute([$username]);
    $user = $stmt->fetch(PDO::FETCH_ASSOC);

    if ($user && password_verify($password, $user['password_hash'])) {
        session_regenerate_id(true);
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['username'] = $user['username'];
        header("Location: index.php");
        exit;
    } else {
        $error = "Invalid credentials";
    }
}
?>
<form method="post">
    <input name="username" placeholder="Username" required>
    <input name="password" type="password" placeholder="Password" required>
    <input type="hidden" name="csrf_token" value="<?= htmlspecialchars($_SESSION['csrf_token']) ?>">
    <button type="submit">Login</button>
</form>
<?php if(!empty($error)) echo "<p style='color:red'>".htmlspecialchars($error)."</p>"; ?>




$stmt->fetch(PDO::FETCH_ASSOC): 


session_regenerate_id(true): 

<?php
session_start();
session_destroy();
header("Location: login.php");



<?php
session_start();
require 'db.php';

if (empty($_SESSION['user_id'])) header("Location: login.php");

$stmt = $pdo->prepare("SELECT id, text FROM todos WHERE user_id=? ORDER BY id ASC");
$stmt->execute([$_SESSION['user_id']]);
$todos = $stmt->fetchAll(PDO::FETCH_ASSOC);

if (empty($_SESSION['csrf_token'])) $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
?>
<h2><?= htmlspecialchars($_SESSION['username']) ?>'s To-Do List</h2>

<ul>
<?php foreach($todos as $t): ?>
  <li><?= htmlspecialchars($t['text']) ?></li>
<?php endforeach; ?>
</ul>

<form method="post" action="add.php">
    <input name="text" placeholder="New todo" required>
    <input type="hidden" name="csrf_token" value="<?= htmlspecialchars($_SESSION['csrf_token']) ?>">
    <button>Add</button>
</form>

<a href="logout.php">Logout</a>

<?php
session_start();
require 'db.php';

if (empty($_SESSION['user_id'])) header("Location: login.php");

$token = $_POST['csrf_token'] ?? '';
if (!hash_equals($_SESSION['csrf_token'] ?? '', $token)) die("Invalid CSRF");

$text = trim($_POST['text'] ?? '');
if ($text !== '') {
    $stmt = $pdo->prepare("INSERT INTO todos (user_id,text) VALUES (?,?)");
    $stmt->execute([$_SESSION['user_id'],$text]);
}

header("Location: index.php");
