const mineflayer = require('mineflayer');
const { pathfinder, Movements, goals } = require('mineflayer-pathfinder');
const pvp = require('mineflayer-pvp').plugin;
const { Vec3 } = require('vec3');

// 1. Configuración del bot
const bot = mineflayer.createBot({
  host: 'infection.fun',
  port: 19132,
  username: 'DarkElo5726Bot'
});

// Registrar los plugins necesarios
bot.loadPlugin(pathfinder);
bot.loadPlugin(pvp);

const PROPIETARIO = 'DarkElo5726'; 

bot.on('spawn', () => {
  console.log('¡El bot ha entrado al servidor correctamente!');
  bot.chat('¡Hola DarkElo5726! Ya estoy en línea y listo.');

  // Configurar los movimientos del pathfinder una vez que cargue el entorno
  const defaultMove = new Movements(bot);
  bot.pathfinder.setMovements(defaultMove);
});

// 2. Escuchar mensajes del chat
bot.on('chat', async (username, message) => {
  if (username === bot.username) return;

  // Verificar si el mensaje viene del propietario autorizado
  if (username === PROPIETARIO) {
    const args = message.split(' ');
    const comando = args[0];

    // --- COMANDO: !ven (Te sigue usando pathfinder) ---
    if (comando === '!ven') {
      const jugador = bot.players[username];
      if (!jugador || !jugador.entity) {
        bot.chat('No te puedo ver, acércate más.');
        return;
      }
      const targetPos = jugador.entity.position;
      bot.chat('¡Voy hacia ti!');
      const p = new goals.GoalFollow(jugador.entity, 1);
      bot.pathfinder.goto(p);
    } 
    
    // --- COMANDO: !atacar (Ataca al mob más cercano) ---
    else if (comando === '!atacar') {
      const entidadCercana = bot.nearestEntity((entity) => {
        // Busca mobs o jugadores que no sean él mismo
        return entity.type === 'mob' || entity.type === 'player';
      });

      if (entidadCercana) {
        bot.chat(`¡Atacando a ${entidadCercana.name || 'objetivo'}!`);
        bot.pvp.attack(entidadCercana);
      } else {
        bot.chat('No veo ninguna criatura cerca para atacar.');
      }
    }

    // --- COMANDO: !detener (Para cancelar acciones de combate o movimiento) ---
    else if (comando === '!detener') {
      bot.pvp.stop();
      bot.pathfinder.stop();
      bot.chat('Acciones detenidas.');
    }

    // --- COMANDO: !poner (Coloca un bloque frente a ti) ---
    else if (comando === '!poner') {
      try {
        // Busca un bloque en el inventario del bot (ej. piedra, tierra)
        const itemBloc = bot.inventory.items().find(item => item.name.includes('stone') || item.name.includes('dirt') || item.name.includes('planks'));
        
        if (!itemBloc) {
          bot.chat('No tengo bloques en mi inventario.');
          return;
        }

        await bot.equip(itemBloc, 'hand');
        
        // Referencia del bloque que está debajo de los pies del bot para construir encima o al lado
        const referenceBlock = bot.blockAt(bot.entity.position.offset(0, -1, 0));
        const jumpOffset = new Vec3(0, 1, 0);

        await bot.placeBlock(referenceBlock, jumpOffset);
        bot.chat('¡Bloque colocado!');
      } catch (err) {
        bot.chat('No pude colocar el bloque.');
        console.log(err);
      }
    }

    // --- COMANDO: !decir ---
    else if (comando === '!decir') {
      const texto = args.slice(1).join(' ');
      if (texto) bot.chat(texto);
    }
  }
});

// Manejar errores de conexión y expulsiones
bot.on('error', (err) => console.log('Ocurrió un error:', err));
bot.on('kicked', (reason) => console.log('El bot fue expulsado por:', reason));
