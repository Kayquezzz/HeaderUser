# HeaderUser

import React, { useState, useRef, useEffect } from 'react';
import "../css/HeaderUser.css"; // Ajuste o caminho se necessário
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap-icons/font/bootstrap-icons.css';
import axios from 'axios';

const recentSearches = [];

const HeaderUser = () => {
    const [searchQuery, setSearchQuery] = useState('');
    const [showRecentTabs, setShowRecentTabs] = useState(false);
    const [sidebarVisible, setSidebarVisible] = useState(false);
    const [modalVisible, setModalVisible] = useState(false);
    const [location, setLocation] = useState('Minha localização');
    const [suggestions, setSuggestions] = useState([]);
    const [chatCardVisible, setChatCardVisible] = useState(false); // Estado para o card de chat
    const searchInputRef = useRef(null);

    useEffect(() => {
        const handleClickOutside = (event) => {
            if (searchInputRef.current && !searchInputRef.current.contains(event.target)) {
                setShowRecentTabs(false);
            }
        };

        document.addEventListener('mousedown', handleClickOutside);

        return () => {
            document.removeEventListener('mousedown', handleClickOutside);
        };
    }, []);

    const handleInputChange = (event) => {
        setSearchQuery(event.target.value);
        if (event.target.value) {
            fetchSuggestions(event.target.value);
        } else {
            setSuggestions([]);
        }
        setShowRecentTabs(event.target.value === '');
    };

    const fetchSuggestions = (input) => {
        const apiKey = 'YOUR_API_KEY'; // Substitua pela sua chave de API
        const url = `https://maps.googleapis.com/maps/api/place/autocomplete/json?input=${input}&key=${apiKey}`;

        axios.get(url)
            .then(response => {
                if (response.data.status === "OK") {
                    setSuggestions(response.data.predictions);
                } else {
                    console.error('Erro ao obter sugestões:', response.data.status);
                }
            })
            .catch(error => {
                console.error('Erro ao buscar sugestões:', error);
            });
    };

    const handleSelectSuggestion = (suggestion) => {
        setLocation(suggestion.description);
        setSearchQuery('');
        setSuggestions([]);
        toggleModal(); // Fecha o modal após selecionar uma sugestão
    };

    const handleSearch = () => {
        if (searchQuery) {
            recentSearches.push(searchQuery);
        }
        setSearchQuery('');
        setShowRecentTabs(false);
    };

    const handleFocus = () => {
        setShowRecentTabs(true);
    };

    const toggleSidebar = () => {
        setSidebarVisible(!sidebarVisible);
    };

    const closeSidebar = () => {
        setSidebarVisible(false);
        setChatCardVisible(false); // Fecha o card de chat ao fechar a sidebar
    };

    const toggleModal = () => {
        setModalVisible(!modalVisible);
    };

    const handleUseLocation = () => {
        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(
                (position) => {
                    const { latitude, longitude } = position.coords;
                    getAddressFromCoordinates(latitude, longitude);
                },
                (error) => {
                    alert('Erro ao acessar a localização: ' + error.message);
                }
            );
        } else {
            alert('Geolocalização não é suportada pelo seu navegador.');
        }
    };

    const getAddressFromCoordinates = (latitude, longitude) => {
        const apiKey = 'AIzaSyDlJ2zlFyM_qx11niZDB10vbIpFjHF---w'; // Substitua pela sua chave de API
        const url = `https://maps.googleapis.com/maps/api/geocode/json?latlng=${latitude},${longitude}&key=${apiKey}`;
        
        axios.get(url)
            .then(response => {
                if (response.data.status === 'OK') {
                    const results = response.data.results;
                    if (results.length > 0) {
                        let street = '';
                        let state = '';
    
                        // Percorra os componentes do endereço para encontrar a rua e o estado
                        results[0].address_components.forEach(component => {
                            if (component.types.includes('route')) {
                                street = component.long_name; // Obtém o nome da rua
                            }
                            if (component.types.includes('administrative_area_level_1')) {
                                state = component.long_name; // Obtém o nome do estado
                            }
                        });
    
                        // Exibir somente a rua e o estado
                        setLocation(`${street}, ${state}`);
                        toggleModal();
                    } else {
                        alert('Não foi possível obter o endereço.');
                    }
                } else {
                    alert('Erro ao obter o endereço.');
                }
            })
            .catch(err => {
                console.error(err);
                alert('Erro ao obter o endereço.');
            });
    };

    const handleChatClick = () => {
        setChatCardVisible(!chatCardVisible); // Alterna a visibilidade do card de chat
    };

    return (
        <div className="header-container" style={{ width: '1384px', height: '900px' }}>
            <div className="header-user">
                <div className="logo-and-nav">
                    <div className="logo">
                        <img src={process.env.PUBLIC_URL + '/organicflaivor.png'} alt="Logo" className="logo-img" />
                    </div>
                    <div className="nav-links">
                        <div className="location-display" onClick={toggleModal}>
                            <i className="bi bi-geo-alt"></i>
                            <span>{location}</span>
                            <i className="bi bi-chevron-down"></i>
                        </div>
                    </div>
                    <div className="menu-hamburguer" onClick={toggleSidebar}>
                        <i className={`bi bi-list ${sidebarVisible ? 'loading' : ''}`}></i>
                    </div>
                </div>
                <div className="search-container" ref={searchInputRef}>
                    <div className="search-bar">
                        <i className="bi bi-search search-icon"></i>
                        <input
                            type="text"
                            placeholder="Buscar..."
                            value={searchQuery}
                            onChange={handleInputChange}
                            onKeyPress={(e) => e.key === 'Enter' && handleSearch()}
                            onFocus={handleFocus}
                        />
                    </div>
                    {showRecentTabs && (
                        <div className={`recent-tabs ${showRecentTabs ? 'show' : ''}`}>
                            {recentSearches.length > 0 ? (
                                <>
                                    <h3>Buscas Recentes</h3>
                                    <ul>
                                        {recentSearches.map((search, index) => (
                                            <li key={index}>
                                                {search}
                                                <i className="bi bi-x-circle" onClick={() => {
                                                    const updatedSearches = recentSearches.filter(item => item !== search);
                                                    localStorage.setItem('recentSearches', JSON.stringify(updatedSearches));
                                                }}></i>
                                            </li>
                                        ))}
                                    </ul>
                                </>
                            ) : (
                                <h3>Você procura por:</h3>
                            )}
                        </div>
                    )}
                </div>
            </div>

            {sidebarVisible && (
                <div className={`sidebar ${sidebarVisible ? 'visible' : ''}`}>
                    <div className="close-btn" onClick={closeSidebar}>
                        <i className="bi bi-x"></i>
                    </div>
                    <div className="logo">
                        <img src={process.env.PUBLIC_URL + '/organicflaivor.png'} alt="Logo" className="sidebar-logo" />
                    </div>
                    <h2>Olá, [Nome do Usuário]</h2>
                    <ul>
                        <li onClick={handleChatClick}>Chat</li>
                        <li>Meus Pedidos</li>
                        <li>Pedidos</li>
                        <li>Configurações</li>
                        <li>Notificações</li>
                    </ul>
                </div>
            )}

            {/* Card Flutuante de Chat */}
            {chatCardVisible && (
                <div className="floating-card" style={{ width: '380px', position: 'fixed', bottom: '500px', left: '250px', backgroundColor: '#fff', boxShadow: '0 2px 10px rgba(0,0,0,0.2)', padding: '20px', borderRadius: '10px', zIndex: 1000 }}>
                    <h3>Conversas</h3>
                    {/* Aqui você pode adicionar conteúdo para as conversas */}
                    <p>Esta é a seção de conversas.</p>
                </div>
            )}

            {/* Modal */}
            {modalVisible && (
                <div className="modal show">
                    <div className="modal-content">
                        <span className="close" onClick={toggleModal}>&times;</span>
                        <img src={process.env.PUBLIC_URL + '/local.png'} alt="Local" style={{ width: '220px', marginBottom: '10px' }} />
                        <p>Onde você quer receber o seu pedido?</p>
                        <div style={{ display: 'flex', alignItems: 'center', width: '100%' }}>
                            <i className="bi bi-search" style={{ marginRight: '10px' }}></i>
                            <input
                                type="text"
                                placeholder="Digite seu endereço..."
                                value={searchQuery}
                                onChange={handleInputChange}
                                onKeyPress={(e) => e.key === 'Enter' && handleSearch()}
                            />
                        </div>
                        <button className="use-location" onClick={handleUseLocation}>Usar Minha Localização</button>
                    </div>
                </div>
            )}
        </div>
    );
};

export default HeaderUser;

CSS:

/* Container principal do header */

.header-user {
    display: flex;
    align-items: center; /* Alinha itens verticalmente */
    justify-content: space-between; /* Espaça itens entre si */
    padding: 10px 20px; /* Adiciona um padding confortável */
    background-color: #fff; /* Cor de fundo do header */
}

.logo {
    flex-shrink: 0; /* Evita que a logo encolha */
}

.logo img {
    width: 75px; /* Ajuste a largura conforme necessário */
    height: auto; /* Mantém a proporção da imagem */
}

.nav-links {
    display: flex; /* Flexbox para os links */
    gap: 15px; /* Espaçamento entre os links */
}

.nav-icons {
    display: flex;
    gap: 15px; /* Espaçamento entre os ícones */
}

.search-container {
    flex-grow: 1; /* Faz a barra de pesquisa ocupar o espaço disponível */
    position: relative; /* Para posicionar o dropdown corretamente */
}

.header-container {
    position: relative;
  }
  
  /* Estilos gerais para o header */
  .header-user {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px 20px;
    background-color: #fff;
    border-bottom: 1px solid #ddd;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
  
  /* Logo e links de navegação */
  .logo-and-nav {
    display: flex;
    align-items: center;
    gap: 20px;
    flex: 1;
  }
  
  .logo {
    flex: 0 1 auto;
  }
  
  .nav-links {
    display: flex;
    gap: 15px;
  }
  
  .nav-link {
    text-decoration: none;
    color: #e8a20d;
    font-weight: bold;
    font-size: 14px;
    transition: color 0.3s ease, transform 0.3s ease;
  }
  
  .nav-link:hover {
    color: #32c810;
    transform: scale(1.1);
  }
  
  /* Contêiner de pesquisa */
  .search-container {
    flex: 2;
    position: relative;
    margin-left: 20px;
  }
  
  .search-bar {
    display: flex;
    align-items: center;
    background-color: #f5f5f5;
    border-radius: 20px;
    padding: 5px;
    transition: all 0.3s ease;
    width: 800px; /* Aumenta a largura da barra de pesquisa */
  }
  
  .search-bar:hover {
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  }
  
  .search-bar input {
    border: none;
    outline: none;
    padding: 5px 10px;
    border-radius: 20px;
    width: 100%;
    background: transparent;
    font-size: 14px;
  }
  
  .search-icon {
    color: #333;
    margin-right: 8px;
  }
  
  /* Recent Tabs */
  .recent-tabs {
    position: absolute;
    top: 100%; /* Posiciona logo abaixo da barra de pesquisa */
    left: 0;
    width: calc(100% - 20px); /* Ajusta a largura do dropdown */
    background-color: #fff;
    border-radius: 10px;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
    z-index: 1000;
    max-height: 300px; /* Ajuste o valor conforme necessário */
    overflow-y: auto;
    opacity: 0;
    transform: translateY(-10px);
    transition: opacity 0.3s ease, transform 0.3s ease;
  }
  
  .recent-tabs.show {
    opacity: 1;
    transform: translateY(0);
  }
  
  .recent-tabs h3 {
    margin: 0;
    padding: 10px;
    font-size: 14px;
    font-weight: bold;
  }
  
  .recent-tabs ul {
    list-style-type: none;
    margin: 0;
    padding: 0;
  }
  
  .recent-tabs li {
    padding: 10px;
    border-bottom: 1px solid #ddd;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .recent-tabs i {
    color: #e74c3c;
    cursor: pointer;
  }
  

.header-user {
  position: relative;
  display: flex;
  align-items: center;
  padding: 10px;
}

.menu-hamburguer {
  position: absolute;
  left: 10px; /* Ajuste a distância da borda esquerda */
  z-index: 1; /* Certifique-se de que o menu esteja acima de outros elementos */
  height: 20px;
}

.logo-and-nav {
  display: flex;
  align-items: center;
  margin-left: 50px; /* Ajuste conforme necessário */
}

.sidebar {
  position: absolute;
  top: 0;
  left: 0;
  width: 250px; /* Ajuste a largura conforme necessário */
  height: 100%;
  background-color: #fff; /* Cor de fundo */
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.2);
  padding: 20px;
  z-index: 1000; /* Certifique-se de que a sidebar esteja acima de outros elementos */
}

.sidebar h2 {
  margin: 0;
  font-size: 18px;
}

.sidebar ul {
  list-style-type: none;
  padding: 0;
}

.sidebar li {
  margin: 10px 0;
  font-size: 16px;
  cursor: pointer; /* Muda o cursor para indicar que é clicável */
}

.loading {
  animation: spin 1s infinite linear; /* Adiciona a animação de rotação */
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.sidebar {
  position: absolute;
  top: 0;
  left: 0;
  width: 250px; /* Ajuste a largura conforme necessário */
  height: 100%;
  background-color: #fff; /* Cor de fundo */
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.2);
  padding: 20px;
  z-index: 1000; /* Certifique-se de que a sidebar esteja acima de outros elementos */
  transform: translateX(-100%); /* Inicialmente oculta */
  transition: transform 0.3s ease; /* Adiciona a animação de deslizamento */
}

.sidebar.visible {
  transform: translateX(0); /* Move para a posição visível */
}

.sidebar h2 {
  margin: 0;
  font-size: 18px;
}

.sidebar ul {
  list-style-type: none;
  padding: 0;
}

.sidebar li {
  margin: 10px 0;
  font-size: 16px;
  cursor: pointer; /* Muda o cursor para indicar que é clicável */
}

.close-btn {
  display: flex;
  justify-content: flex-end; /* Alinha o botão à direita */
  cursor: pointer; /* Muda o cursor para indicar que é clicável */
  font-size: 20px; /* Tamanho do ícone */
  margin-bottom: 20px; /* Espaço abaixo do botão */
}

.close-btn i {
  color: #e74c3c; /* Cor do ícone */
}

.location-display {
  display: flex;
  align-items: center;
  cursor: pointer; /* Indica que é clicável */
  white-space: nowrap; /* Impede a quebra de linha */
}

.location-display span {
  margin: 0 8px; /* Espaçamento ao redor do texto */
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1001; /* Acima da sidebar */
  animation: fadeIn 0.3s ease; /* Animação de aparecimento */
}

.modal-content {
  width: 694px;
  height: 576px;
  background: white;
  border-radius: 8px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  transform: translateY(-20px); /* Começa um pouco acima */
  animation: slideDown 0.3s ease; /* Animação de deslizamento */
}

@keyframes fadeIn {
  from {
      opacity: 0;
  }
  to {
      opacity: 1;
  }
}

@keyframes slideDown {
  from {
      transform: translateY(-20px);
  }
  to {
      transform: translateY(0);
  }
}

.modal {
  display: none; /* Inicialmente escondido */
  position: fixed;
  z-index: 1000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  justify-content: center;
  align-items: center;
  opacity: 0; /* Começa invisível */
  transform: translateY(-100%); /* Começa fora da tela */
  transition: opacity 0.3s ease, transform 0.3s ease; /* Transições */
}

.modal.show {
  display: flex; /* Exibe o modal quando ativo */
  opacity: 1; /* Torna visível */
  transform: translateY(0); /* Move para a posição visível */
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
  width: 80%;
  max-width: 694px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.close {
  cursor: pointer;
  float: right;
  font-size: 20px;
  color: #e74c3c;
}

.search-bar-modal {
  display: flex;
  align-items: center;
  margin-top: 20px; /* Ajuste o espaço acima conforme necessário */
}

.search-bar-modal input {
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 10px;
  width: 100%; /* Ajuste conforme necessário */
  margin-left: 10px; /* Espaço entre o ícone e o input */
}

.search-icon {
  color: #333; /* Cor do ícone */
}

.search-bar-modal {
  display: flex;
  align-items: center;
  margin-top: 20px; /* Ajuste o espaço acima conforme necessário */
}

.search-bar-modal input {
  border: none; /* Remove as bordas */
  border-radius: 4px; /* Você pode manter isso se quiser bordas arredondadas */
  padding: 10px;
  width: 100%; /* Ocupar todo o espaço disponível */
  margin-left: 10px; /* Espaço entre o ícone e o input */
  background-color: #f5f5f5; /* Cor de fundo opcional */
  font-size: 16px; /* Aumenta o tamanho da fonte */
}

.search-icon {
  color: #333; /* Cor do ícone */
  font-size: 20px; /* Ajuste o tamanho do ícone se necessário */
}

.modal-content {
  /* outros estilos já existentes */
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}




