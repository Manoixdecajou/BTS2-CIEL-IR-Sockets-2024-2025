# TP Programmation Réseau : Création d'un jeu de Tic-Tac-Toe en réseau

## Objectifs pédagogiques
- Comprendre les différences entre UDP et TCP
- Maîtriser la programmation des sockets avec SFML
- Implémenter un protocole de découverte de serveurs
- Gérer une connexion client/serveur
- Manipuler des paquets réseau

## Partie 1 : Découverte des serveurs (UDP)

### Exercice 1.1 : Initialisation du socket UDP
Complétez la méthode `NetworkDiscovery::Init()` pour :
- Créer un socket UDP
- Le lier à un port
- Gérer les erreurs de liaison

```cpp
bool NetworkDiscovery::Init()
{
    uint16_t port = NetworkPort;
	// 1. Essayer de lier le socket au port NetworkPort
	sf::Socket::Status status;
	status = _socket.bind(port);
	
	if (status  != sf::Socket::Done)
	{
		while (status != sf::Socket::Done)
		{
			// 2. Si échec, essayer les ports suivants jusqu'à succès
			port += 1;
			status = _socket.bind(port);
		}
			
	}
	// 3. Ajouter le socket au sélecteur
	_socketSelector.add(_socket);
	return true;
}
```

### Exercice 1.2 : Broadcast des serveurs
Complétez la méthode `NetworkDiscovery::Update()` pour implémenter le broadcast :

```cpp
void NetworkDiscovery::Update()
{
    uint64_t nowMs = GetTimeMs();

	if(_isBroadcastEnabled)
	{
		// 1. Vérifier si l'écart de temps entre maintenant et la dernière déclaration de temps est supérieure ou égale à DeclareGameServerDelayMs
		if ((nowMs - _lastDeclareGameServerTimeMs ) >= DeclareGameServerDelayMs)
		{
			// 2. Créer un paquet avec MagicPacket et _localServerName
			sf::Packet packet;
			packet << MagicPacket << _localServerName;

			// 3. Envoyer le paquet en broadcast
			_socket.send(packet,sf::IpAddress::Broadcast);
		}
	}

    // Le reste du code est fourni...

```

## Partie 2 : Connexion TCP

### Exercice 2.1 : Configuration du serveur
Complétez la méthode `NetworkGame::HostGame()` :

```cpp
bool NetworkGame::HostGame()
{
    	sf::Socket::Status status;
	// 1. Configurer le listener TCP sur NetworkPort
	status = _listener.listen(NetworkPort)
	if (status == sf::Socket::Done)
	{
		// 2. Ajouter le listener au sélecteur
		_selector.add(_listener);

		// 3. Définir _isServer à true
		_isServer = true;
	}
    return true;
}
```

### Exercice 2.2 : Attente de connexion
Complétez la méthode `NetworkGame::WaitingAnOpponent()` :

```cpp
bool NetworkGame::WaitingAnOpponent()
{
    sf::TcpSocket* client = new sf::TcpSocket;

	// 1. Vérifier si une connexion est en attente avec le sélecteur
	_listener.accept(*_selector);

	if (_selector.isReady(_listener))
	{
		// 2. Accepter la connexion
		_listener.accept(_socket);
		

	}
	else
	{
		return false;
	}

	// 3. Envoyer le nom du joueur local

	std::string name = &GetLocalPlayerName();
	sf::Packet packet;
	packet << MagicPacket << name;
	_socket.send(packet, _socket.getRemoteAddress());
	// 4. Configurer la connexion
	_isConnectionEstablish = true;

	return true;
}
```

## Partie 3 : Protocole de jeu

### Exercice 3.1 : Envoi d'un coup
Implémentez la méthode `NetworkGame::SendMove()` :

```cpp
bool NetworkGame::SendMove(int row, int col)
{
    // À compléter :
    // 1. Créer un paquet avec MagicPacket et PacketType::Move
    // 2. Ajouter les coordonnées row et col
    // 3. Envoyer le paquet
    return true;
}
```

### Exercice 3.2 : Réception d'un coup
Complétez la méthode `NetworkGame::TryToReceivePacket()` :

```cpp
sf::Socket::Status NetworkGame::TryToReceivePacket(PacketType packetTypeExpect)
{
    // À compléter :
    // 1. Vérifier si des données sont disponibles
    // 2. Recevoir le paquet
    // 3. Vérifier le MagicPacket
    // 4. Vérifier le type de paquet
    return sf::Socket::Done;
}
```

## Exercices bonus

### Bonus 1 : Gestion des timeouts
Ajoutez un système de timeout pour détecter la déconnexion d'un joueur.

### Bonus 2 : Chat entre joueurs
Ajoutez un système de chat entre les joueurs en créant un nouveau type de paquet.

## Conseils pour le debug

- Utilisez `std::cout` pour tracer les événements réseau
- Testez d'abord en local (127.0.0.1)
- Vérifiez les pare-feu pour le broadcast
- Utilisez Wireshark pour observer les échanges

## Ressources supplémentaires
- Documentation SFML : sfml-dev.org/documentation/2.5.1/
- RFC sur UDP : rfc768
- RFC sur TCP : rfc793
