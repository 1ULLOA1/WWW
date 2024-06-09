mkdir anti-estafas
cd anti-estafas

npm init -y

npm install express ejs body-parser multer sequelize sqlite3
mkdir public public/css views uploads
touch public/css/styles.css views/index.ejs views/report.ejs views/reports.ejs views/resources.ejs views/contact.ejs app.js
mkdir models
touch models/index.js
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: 'database.sqlite'
});

const Report = sequelize.define('Report', {
  nombre: {
    type: DataTypes.STRING,
    allowNull: true
  },
  email: {
    type: DataTypes.STRING,
    allowNull: true
  },
  titulo: {
    type: DataTypes.STRING,
    allowNull: false
  },
  descripcion: {
    type: DataTypes.TEXT,
    allowNull: false
  },
  fecha: {
    type: DataTypes.DATE,
    allowNull: true
  },
  cantidad: {
    type: DataTypes.FLOAT,
    allowNull: true
  },
  archivo: {
    type: DataTypes.STRING,
    allowNull: true
  }
});

const Contact = sequelize.define('Contact', {
  nombre: {
    type: DataTypes.STRING,
    allowNull: false
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false
  },
  mensaje: {
    type: DataTypes.TEXT,
    allowNull: false
  }
});

sequelize.sync();

module.exports = { sequelize, Report, Contact };
const express = require('express');
const bodyParser = require('body-parser');
const multer = require('multer');
const path = require('path');
const { sequelize, Report, Contact } = require('./models');

const app = express();
const port = 3000;

app.set('view engine', 'ejs');

app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static(path.join(__dirname, 'public')));

const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, 'uploads/');
  },
  filename: function (req, file, cb) {
    cb(null, Date.now() + path.extname(file.originalname));
  }
});
const upload = multer({ storage: storage });

app.get('/', async (req, res) => {
  res.render('index');
});

app.get('/report', (req, res) => {
  res.render('report');
});

app.post('/submit-report', upload.single('archivo'), async (req, res) => {
  try {
    await Report.create({
      nombre: req.body.nombre,
      email: req.body.email,
      titulo: req.body.titulo,
      descripcion: req.body.descripcion,
      fecha: req.body.fecha,
      cantidad: req.body.cantidad,
      archivo: req.file ? req.file.filename : null
    });
    res.redirect('/reports');
  } catch (error) {
    res.status(500).send('Error al enviar el reporte');
  }
});

app.get('/reports', async (req, res) => {
  const reports = await Report.findAll();
  res.render('reports', { reports });
});

app.get('/resources', (req, res) => {
  res.render('resources');
});

app.get('/contact', (req, res) => {
  res.render('contact');
});

app.post('/submit-contact', async (req, res) => {
  try {
    await Contact.create({
      nombre: req.body.nombre,
      email: req.body.email,
      mensaje: req.body.mensaje
    });
    res.redirect('/');
  } catch (error) {
    res.status(500).send('Error al enviar el mensaje');
  }
});

app.listen(port, () => {
  console.log(`Servidor escuchando en http://localhost:${port}`);
});
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Página Principal</title>
  <link rel="stylesheet" href="/css/styles.css">
</head>
<body>
  <header>
    <h1>Bienvenidos a Anti-Estafas</h1>
    <p>El lugar donde puedes compartir tus experiencias y ayudar a otros a evitar estafas.</p>
  </header>
  <section id="destacados">
    <h2>Estafas Recientes</h2>
    <!-- Aquí irán los resúmenes de las estafas recientes -->
  </section>
  <footer>
    <p>&copy; 2024 Anti-Estafas. Todos los derechos reservados.</p>
  </footer>
</body>
</html>
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Formulario de Reporte</title>
  <link rel="stylesheet" href="/css/styles.css">
</head>
<body>
  <header>
    <h1>Reportar una Estafa</h1>
  </header>
  <section>
    <form action="/submit-report" method="post" enctype="multipart/form-data">
      <label for="nombre">Nombre (opcional):</label>
      <input type="text" id="nombre" name="nombre">
      
      <label for="email">Correo Electrónico (opcional):</label>
      <input type="email" id="email" name="email">
      
      <label for="titulo">Título de la Estafa:</label>
      <input type="text" id="titulo" name="titulo" required>
      
      <label for="descripcion">Descripción de la Estafa:</label>
      <textarea id="descripcion" name="descripcion" required></textarea>
      
      <label for="fecha">Fecha de la Estafa:</label>
      <input type="date" id="fecha" name="fecha">
      
      <label for="cantidad">Cantidad Perdida (opcional):</label>
      <input type="number" id="cantidad" name="cantidad">
      
      <label for="archivo">Archivo Adjunto (opcional):</label>
      <input type="file" id="archivo" name="archivo">
      
      <button type="submit">Enviar Reporte</button>
    </form>
  </section>
  <footer>
    <p>&copy; 2024 Anti-Estafas. Todos los derechos reservados.</p>
  </footer>
</body>
</html>
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Lista de Reportes</title>
  <link rel="stylesheet" href="/css/styles.css">
</head>
<body>
  <header>
    <h1>Reportes de Estafas</h1>
  </header>
  <section>
    <div id="reportes">
      <% reports.forEach(report => { %>
        <article>
          <h2><%= report.titulo %></h2>
          <p><%= report.descripcion %></p>
          <p><small>Reportado por: <%= report.nombre || 'Anónimo' %> el <%= report.fecha ? new Date(report.fecha).toLocaleDateString() : 'Fecha no especificada' %></small></p>
        </article>
      <% }) %>
    </div>
  </section>
  <footer>
    <p>&copy; 2024 Anti-Estafas. Todos los derechos reservados.</p>
  </footer>
</body>
</html>
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Recursos y Consejos</title>
  <link rel="stylesheet" href="/css/styles.css">
</head>
<body>
  <header>
    <h1>Recursos y Consejos para Evitar Estafas</h1>
  </header>
  <section>
    <h2>Cómo Protegerse de las Estafas</h2>
    <p><!-- Información sobre cómo protegerse --></p>
  </section>
  <footer>
    <p>&copy; 2024 Anti-Estafas. Todos los derechos reservados.</p>
  </footer>
</body>
</html>
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Contacto</title>
  <link rel="stylesheet" href="/css/styles.css">
</head>
<body>
  <header>
    <h1>Contacto</h1>
  </header>
  <section>
    <form action="/submit-contact" method="post">
      <label for="nombre">Nombre:</label>
      <input type="text" id="nombre" name="nombre" required>
      
      <label for="email">Correo Electrónico:</label>
      <input type="email" id="email" name="email" required>
      
      <label for="mensaje">Mensaje:</label>
      <textarea id="mensaje" name="mensaje" required></textarea>
      
      <button type="submit">Enviar Mensaje</button>
    </form>
  </section>
  <footer>
    <p>&copy; 2024 Anti-Estafas. Todos los derechos reservados.</p>
  </footer>
</body>
</html>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  background-color: #f4f4f4;
}

header {
  background-color: #333;
  color: #fff;
  padding: 1em 0;
  text-align: center;
}

header h1 {
  margin: 0;
}

section {
  padding: 2em;
}

form {
  background: #fff;
  padding: 2em;
  margin: 2em 0;
  box-shadow: 0 0 1em #ccc;
}

form label {
  display: block;
  margin: 1em 0 0.5em;
}

form input, form textarea {
  width: 100%;
  padding: 0.5em;
  margin-bottom: 1em;
  border: 1px solid #ccc;
}

form button {
  background: #333;
  color: #fff;
  padding: 1em 2em;
  border: none;
  cursor: pointer;
}

form button:hover {
  background: #555;
}

footer {
  background: #333;
  color: #fff;
  text-align: center;
  padding: 1em 0;
  position: fixed;
  bottom: 0;
  width: 100%;
}
node app.js
heroku login
heroku create
git init
git add .
git commit -m "Initial commit"
heroku git:remote -a <nombre-de-tu-app>
git push heroku master
web: node app.js
