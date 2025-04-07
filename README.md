<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ProjetoPRO - Gerenciamento de Projetos</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        primary: {
                            50: '#fee2e2',
                            100: '#fecaca',
                            200: '#fca5a5',
                            300: '#f87171',
                            400: '#ef4444',
                            500: '#dc2626',
                            600: '#b91c1c',
                            700: '#991b1b',
                            800: '#7f1d1d',
                            900: '#6f1a1a'
                        }
                    }
                }
            }
        }

        // Check for dark mode preference
        if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
            document.documentElement.classList.add('dark');
        }
        window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', event => {
            if (event.matches) {
                document.documentElement.classList.add('dark');
            } else {
                document.documentElement.classList.remove('dark');
            }
        });
    </script>
    <style>
        .drag-item {
            cursor: grab;
            transition: transform 0.2s, box-shadow 0.2s;
        }
        .drag-item:active {
            cursor: grabbing;
            transform: scale(1.02);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        .drop-zone {
            transition: background-color 0.2s;
        }
        .drop-zone.drag-over {
            background-color: rgba(220, 38, 38, 0.1);
        }
        .task-move-animation {
            transition: all 0.3s ease;
        }
        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: transparent;
        }
        ::-webkit-scrollbar-thumb {
            background: #CBD5E1;
            border-radius: 3px;
        }
        .dark ::-webkit-scrollbar-thumb {
            background: #475569;
        }
        /* Toast notification */
        .toast {
            animation: slideIn 0.3s, fadeOut 0.5s 2.5s forwards;
            max-width: 350px;
        }
        @keyframes slideIn {
            from { transform: translateY(-100%); opacity: 0; }
            to { transform: translateY(0); opacity: 1; }
        }
        @keyframes fadeOut {
            from { opacity: 1; }
            to { opacity: 0; visibility: hidden; }
        }
        /* Calendar styles */
        .calendar-grid {
            display: flex;
            flex-direction: column;
        }
        .calendar-day {
            aspect-ratio: 1;
            display: flex;
            flex-direction: column;
            border-radius: 0.25rem;
            overflow: hidden;
            cursor: pointer;
            position: relative;
            transition: all 0.2s ease;
        }
        .calendar-day.today {
            border: 2px solid;
        }
        .calendar-day.selected {
            background-color: rgba(220, 38, 38, 0.15);
            box-shadow: 0 0 0 2px rgba(220, 38, 38, 0.5);
            z-index: 4;
        }
        .calendar-day.has-events {
            background-color: rgba(220, 38, 38, 0.05);
        }
        .calendar-day.has-events .event-indicator {
            position: absolute;
            bottom: 4px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            gap: 2px;
        }
        .calendar-day.has-events .event-indicator .dot {
            height: 4px;
            width: 4px;
            border-radius: 50%;
        }
        .calendar-day .event-preview {
            position: absolute;
            bottom: 100%;
            left: 50%;
            transform: translateX(-50%);
            background-color: white;
            border: 1px solid #e5e7eb;
            border-radius: 0.25rem;
            padding: 0.5rem;
            width: 150px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            z-index: 10;
            display: none;
        }
        .dark .calendar-day .event-preview {
            background-color: #374151;
            border-color: #4b5563;
        }
        .calendar-day:hover .event-preview {
            display: block;
        }
        .calendar-day:hover {
            background-color: rgba(0, 0, 0, 0.05);
            transform: scale(1.05);
            z-index: 5;
        }
        .dark .calendar-day:hover {
            background-color: rgba(255, 255, 255, 0.05);
        }
        .calendar-day.other-month {
            opacity: 0.4;
        }
        .day-events {
            max-height: 100%;
            overflow: hidden;
        }
        .calendar-day .day-events .event-badge {
            margin-top: 2px;
            padding: 1px 3px;
            border-radius: 2px;
            font-size: 0.6rem;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            opacity: 0.8;
        }

        /* Timeline lanes */
        .timeline-lane {
            position: relative;
            height: 60px;
            margin-bottom: 5px;
            border-bottom: 1px dashed rgba(200, 200, 200, 0.3);
        }
        .timeline-item {
            transition: all 0.2s ease;
        }
        .timeline-item .timeline-card {
            transform: scale(0.85);
            opacity: 0.9;
            transition: all 0.2s ease;
            max-width: 250px;
        }
        .timeline-item:hover .timeline-card {
            transform: scale(1);
            opacity: 1;
            z-index: 10;
        }
        .timeline-item .timeline-card-full-content {
            display: none;
            max-height: 0;
            overflow: hidden;
            transition: max-height 0.3s ease;
        }
        .timeline-item.expanded .timeline-card-full-content {
            display: block;
            max-height: 200px;
        }
        .timeline-item .timeline-compact-view {
            display: block;
        }
        .timeline-item.expanded .timeline-compact-view {
            display: none;
        }
        .timeline-item-wrapper {
            position: absolute;
        }
        /* Override to prevent overlapping */
        .timeline-lane-item {
            margin-top: 10px;
            position: relative;
            transition: all 0.2s ease;
        }
        .timeline-lane-item:nth-child(even) {
            margin-top: 30px;
        }
        .timeline-lane-item:nth-child(3n) {
            margin-top: 50px;
        }

        .calendar-event-item {
            cursor: pointer;
            transition: all 0.2s ease;
        }
        .calendar-event-item:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body class="bg-white dark:bg-gray-900 text-gray-800 dark:text-gray-200 min-h-screen flex flex-col">
    <!-- Toast notifications container -->
    <div id="toastContainer" class="fixed top-4 right-4 z-50 flex flex-col gap-2"></div>

    <!-- Loading Overlay - oculto por padrão agora-->
    <div id="loadingOverlay" class="fixed inset-0 bg-black bg-opacity-50 z-50 flex items-center justify-center hidden">
        <div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-xl text-center">
            <div class="inline-block animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-primary-500 mb-4"></div>
            <h2 class="text-xl font-semibold">Carregando dados...</h2>
            <p class="text-gray-500 dark:text-gray-400 mt-2">Por favor, aguarde enquanto carregamos seu projeto.</p>
        </div>
    </div>

    <!-- Header/Navigation -->
    <header class="bg-white dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700 shadow-sm sticky top-0 z-10">
        <div class="container mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <i class="fas fa-project-diagram text-primary-500 text-2xl"></i>
                <h1 class="text-xl font-bold">ProjetoPRO</h1>
            </div>
            <div class="flex items-center space-x-3">
                <button id="importBtn" class="text-sm px-3 py-1.5 rounded-md bg-gray-100 dark:bg-gray-700 hover:bg-gray-200 dark:hover:bg-gray-600">
                    <i class="fas fa-file-import mr-1"></i> Importar
                </button>
                <button id="exportBtn" class="text-sm px-3 py-1.5 rounded-md bg-gray-100 dark:bg-gray-700 hover:bg-gray-200 dark:hover:bg-gray-600">
                    <i class="fas fa-file-export mr-1"></i> Exportar
                </button>
                <div class="relative">
                    <button id="userMenuBtn" class="flex items-center focus:outline-none">
                        <div class="w-8 h-8 rounded-full bg-primary-500 flex items-center justify-center text-white">
                            <i class="fas fa-user"></i>
                        </div>
                    </button>
                    <div id="userMenu" class="absolute right-0 mt-2 w-48 bg-white dark:bg-gray-800 rounded-md shadow-lg py-1 hidden">
                        <a href="#" id="settingsBtn" class="block px-4 py-2 text-sm hover:bg-gray-100 dark:hover:bg-gray-700">
                            <i class="fas fa-cog mr-2"></i> Configurações
                        </a>
                        <a href="#" id="helpBtn" class="block px-4 py-2 text-sm hover:bg-gray-100 dark:hover:bg-gray-700">
                            <i class="fas fa-question-circle mr-2"></i> Ajuda
                        </a>
                        <a href="#" id="toggleDarkMode" class="block px-4 py-2 text-sm hover:bg-gray-100 dark:hover:bg-gray-700">
                            <i class="fas fa-moon mr-2 dark:hidden"></i>
                            <i class="fas fa-sun mr-2 hidden dark:inline-block"></i>
                            <span class="dark:hidden">Modo Escuro</span>
                            <span class="hidden dark:inline-block">Modo Claro</span>
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <div class="flex-1 flex">
        <!-- Sidebar -->
        <aside class="w-16 md:w-64 bg-white dark:bg-gray-800 border-r border-gray-200 dark:border-gray-700 flex-shrink-0">
            <div class="h-full flex flex-col">
                <nav class="flex-1 py-4 px-2">
                    <ul class="space-y-1">
                        <li>
                            <a href="#dashboard" class="nav-link flex items-center p-2 rounded-md text-primary-500 bg-primary-50 dark:bg-primary-900/30">
                                <i class="fas fa-home w-6 text-center md:mr-2"></i>
                                <span class="hidden md:block">Dashboard</span>
                            </a>
                        </li>
                        <li>
                            <a href="#projects" class="nav-link flex items-center p-2 rounded-md hover:bg-gray-100 dark:hover:bg-gray-700">
                                <i class="fas fa-folder w-6 text-center md:mr-2"></i>
                                <span class="hidden md:block">Projetos</span>
                            </a>
                        </li>
                        <li>
                            <a href="#tasks" class="nav-link flex items-center p-2 rounded-md hover:bg-gray-100 dark:hover:bg-gray-700">
                                <i class="fas fa-tasks w-6 text-center md:mr-2"></i>
                                <span class="hidden md:block">Tarefas</span>
                            </a>
                        </li>
                        <li>
                            <a href="#kanban" class="nav-link flex items-center p-2 rounded-md hover:bg-gray-100 dark:hover:bg-gray-700">
                                <i class="fas fa-columns w-6 text-center md:mr-2"></i>
                                <span class="hidden md:block">Kanban</span>
                            </a>
                        </li>
                        <li>
                            <a href="#timeline" class="nav-link flex items-center p-2 rounded-md hover:bg-gray-100 dark:hover:bg-gray-700">
                                <i class="fas fa-calendar-alt w-6 text-center md:mr-2"></i>
                                <span class="hidden md:block">Cronograma</span>
                            </a>
                        </li>
                        <li>
                            <a href="#logs" class="nav-link flex items-center p-2 rounded-md hover:bg-gray-100 dark:hover:bg-gray-700">
                                <i class="fas fa-history w-6 text-center md:mr-2"></i>
                                <span class="hidden md:block">Registros</span>
                            </a>
                        </li>
                        <li>
                            <a href="#ideas" class="nav-link flex items-center p-2 rounded-md hover:bg-gray-100 dark:hover:bg-gray-700">
                                <i class="fas fa-lightbulb w-6 text-center md:mr-2"></i>
                                <span class="hidden md:block">Ideias</span>
                            </a>
                        </li>
                    </ul>
                </nav>
                <div class="p-4 border-t border-gray-200 dark:border-gray-700 hidden md:block">
                    <div class="text-xs text-gray-500 dark:text-gray-400">
                        Armazenamento
                        <div class="mt-1 h-2 w-full bg-gray-200 dark:bg-gray-700 rounded-full overflow-hidden">
                            <div id="storageIndicator" class="h-full bg-primary-500" style="width: 0%"></div>
                        </div>
                        <div class="mt-1 flex justify-between text-xs">
                            <span id="storageUsed">0 KB</span>
                            <span>Sem limite</span>
                        </div>
                    </div>
                </div>
            </div>
        </aside>

        <!-- Main Content Area -->
        <main class="flex-1 overflow-x-hidden overflow-y-auto bg-gray-50 dark:bg-gray-900 p-4">
            <!-- Page content will be loaded here -->
            <div id="pageContent">
                <!-- Os templates das views serão injetados aqui dinamicamente -->
            </div>
        </main>
    </div>

    <!-- MODALS -->
    <!-- New Project Modal -->
    <div id="newProjectModal" class="fixed inset-0 z-50 overflow-y-auto hidden">
        <div class="flex items-center justify-center min-h-screen px-4">
            <div class="fixed inset-0 bg-black bg-opacity-50 transition-opacity" id="newProjectModalBackdrop"></div>
            <div class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-xl transform transition-all max-w-lg w-full">
                <div class="px-6 py-4 border-b border-gray-200 dark:border-gray-700">
                    <h3 class="text-lg font-semibold">Novo Projeto</h3>
                </div>
                <form id="newProjectForm" class="p-6">
                    <div class="space-y-4">
                        <div>
                            <label for="projectName" class="block text-sm font-medium mb-1">Nome do Projeto (O quê?)</label>
                            <input type="text" id="projectName" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" required>
                        </div>
                        <div>
                            <label for="projectDescription" class="block text-sm font-medium mb-1">Descrição</label>
                            <textarea id="projectDescription" rows="3" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700"></textarea>
                        </div>

                        <!-- 5W2H Fields -->
                        <div>
                            <label for="projectWhy" class="block text-sm font-medium mb-1">Por quê? (Why)</label>
                            <textarea id="projectWhy" rows="2" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" placeholder="Objetivo e justificativa do projeto"></textarea>
                        </div>
                        <div>
                            <label for="projectWhere" class="block text-sm font-medium mb-1">Onde? (Where)</label>
                            <input type="text" id="projectWhere" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" placeholder="Local de execução">
                        </div>
                        <div>
                            <label for="projectWho" class="block text-sm font-medium mb-1">Quem? (Who)</label>
                            <input type="text" id="projectWho" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" placeholder="Responsáveis pelo projeto">
                        </div>
                        <div>
                            <label for="projectHow" class="block text-sm font-medium mb-1">Como? (How)</label>
                            <textarea id="projectHow" rows="2" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" placeholder="Método e processos utilizados"></textarea>
                        </div>
                        <div>
                            <label for="projectHowMuch" class="block text-sm font-medium mb-1">Quanto custa? (How much)</label>
                            <input type="text" id="projectHowMuch" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" placeholder="Custo estimado">
                        </div>

                        <div class="grid grid-cols-2 gap-4">
                            <div>
                                <label for="projectStartDate" class="block text-sm font-medium mb-1">Data de Início (When)</label>
                                <input type="date" id="projectStartDate" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700">
                            </div>
                            <div>
                                <label for="projectDueDate" class="block text-sm font-medium mb-1">Data de Conclusão</label>
                                <input type="date" id="projectDueDate" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700">
                            </div>
                        </div>
                        <div>
                            <label for="projectPriority" class="block text-sm font-medium mb-1">Prioridade</label>
                            <select id="projectPriority" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700">
                                <option value="low">Baixa</option>
                                <option value="medium" selected>Média</option>
                                <option value="high">Alta</option>
                            </select>
                        </div>
                    </div>
                    <div class="mt-6 flex justify-end space-x-3">
                        <button type="button" id="cancelNewProject" class="px-4 py-2 text-sm font-medium text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-md">Cancelar</button>
                        <button type="submit" class="px-4 py-2 text-sm font-medium text-white bg-primary-500 hover:bg-primary-600 rounded-md shadow-sm">Criar Projeto</button>
                    </div>
                </form>
            </div>
        </div>
    </div>

    <!-- Project Detail Modal -->
    <div id="projectDetailModal" class="fixed inset-0 z-50 overflow-y-auto hidden">
        <div class="flex items-center justify-center min-h-screen px-4">
            <div class="fixed inset-0 bg-black bg-opacity-50 transition-opacity" id="projectDetailModalBackdrop"></div>
            <div class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-xl transform transition-all max-w-4xl w-full">
                <div class="px-6 py-4 border-b border-gray-200 dark:border-gray-700 flex justify-between items-center">
                    <h3 class="text-lg font-semibold" id="projectDetailTitle">Detalhes do Projeto</h3>
                    <button id="closeProjectDetail" class="text-gray-400 hover:text-gray-500 focus:outline-none">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
                <div id="projectDetailContent" class="p-6">
                    <!-- O conteúdo será carregado dinamicamente -->
                </div>
            </div>
        </div>
    </div>

    <!-- New Task Modal -->
    <div id="taskModal" class="fixed inset-0 z-50 overflow-y-auto hidden">
        <div class="flex items-center justify-center min-h-screen px-4">
            <div class="fixed inset-0 bg-black bg-opacity-50 transition-opacity" id="taskModalBackdrop"></div>
            <div class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-xl transform transition-all max-w-lg w-full">
                <div class="px-6 py-4 border-b border-gray-200 dark:border-gray-700 flex justify-between items-center">
                    <h3 class="text-lg font-semibold" id="taskModalTitle">Nova Tarefa</h3>
                    <button id="closeTaskModal" class="text-gray-400 hover:text-gray-500 focus:outline-none">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
                <form id="taskForm" class="p-6">
                    <input type="hidden" id="taskId">
                    <div class="space-y-4">
                        <div>
                            <label for="taskName" class="block text-sm font-medium mb-1">Nome da Tarefa</label>
                            <input type="text" id="taskName" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" required>
                        </div>
                        <div>
                            <label for="taskDescription" class="block text-sm font-medium mb-1">Descrição</label>
                            <textarea id="taskDescription" rows="3" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700"></textarea>
                        </div>
                        <div>
                            <label for="taskProjectId" class="block text-sm font-medium mb-1">Projeto</label>
                            <select id="taskProjectId" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" required>
                                <!-- Opções serão carregadas dinamicamente -->
                            </select>
                        </div>
                        <div>
                            <label for="taskStatus" class="block text-sm font-medium mb-1">Status</label>
                            <select id="taskStatus" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700">
                                <option value="todo">A Fazer</option>
                                <option value="in_progress">Em Progresso</option>
                                <option value="completed">Concluída</option>
                            </select>
                        </div>
                        <div>
                            <label for="taskPriority" class="block text-sm font-medium mb-1">Prioridade</label>
                            <select id="taskPriority" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700">
                                <option value="low">Baixa</option>
                                <option value="medium" selected>Média</option>
                                <option value="high">Alta</option>
                            </select>
                        </div>
                        <div>
                            <label for="taskDueDate" class="block text-sm font-medium mb-1">Data de Conclusão</label>
                            <input type="date" id="taskDueDate" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700">
                        </div>
                        
                        <!-- 5W2H Fields específicos da tarefa -->
                        <div>
                            <label for="taskWho" class="block text-sm font-medium mb-1">Responsável (Who)</label>
                            <input type="text" id="taskWho" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" placeholder="Responsável pela tarefa">
                        </div>
                        <div>
                            <label for="taskHow" class="block text-sm font-medium mb-1">Como fazer (How)</label>
                            <textarea id="taskHow" rows="2" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" placeholder="Procedimento para realizar a tarefa"></textarea>
                        </div>
                    </div>
                    <div class="mt-6 flex justify-end space-x-3">
                        <button type="button" id="cancelTask" class="px-4 py-2 text-sm font-medium text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-md">Cancelar</button>
                        <button type="submit" class="px-4 py-2 text-sm font-medium text-white bg-primary-500 hover:bg-primary-600 rounded-md shadow-sm">Salvar</button>
                    </div>
                </form>
            </div>
        </div>
    </div>

    <!-- New Idea Modal -->
    <div id="ideaModal" class="fixed inset-0 z-50 overflow-y-auto hidden">
        <div class="flex items-center justify-center min-h-screen px-4">
            <div class="fixed inset-0 bg-black bg-opacity-50 transition-opacity" id="ideaModalBackdrop"></div>
            <div class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-xl transform transition-all max-w-lg w-full">
                <div class="px-6 py-4 border-b border-gray-200 dark:border-gray-700 flex justify-between items-center">
                    <h3 class="text-lg font-semibold" id="ideaModalTitle">Nova Ideia</h3>
                    <button id="closeIdeaModal" class="text-gray-400 hover:text-gray-500 focus:outline-none">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
                <form id="ideaForm" class="p-6">
                    <input type="hidden" id="ideaId">
                    <div class="space-y-4">
                        <div>
                            <label for="ideaTitle" class="block text-sm font-medium mb-1">Título da Ideia</label>
                            <input type="text" id="ideaTitle" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" required>
                        </div>
                        <div>
                            <label for="ideaDescription" class="block text-sm font-medium mb-1">Descrição</label>
                            <textarea id="ideaDescription" rows="4" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" required></textarea>
                        </div>
                        <div>
                            <label for="ideaCategory" class="block text-sm font-medium mb-1">Categoria</label>
                            <select id="ideaCategory" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700">
                                <option value="feature">Nova funcionalidade</option>
                                <option value="improvement">Melhoria</option>
                                <option value="project">Projeto</option>
                                <option value="process">Processo</option>
                                <option value="other">Outra</option>
                            </select>
                        </div>
                        <div>
                            <label for="ideaTags" class="block text-sm font-medium mb-1">Tags (separadas por vírgula)</label>
                            <input type="text" id="ideaTags" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700" placeholder="Ex: inovação, estratégia, automação">
                        </div>
                    </div>
                    <div class="mt-6 flex justify-end space-x-3">
                        <button type="button" id="cancelIdea" class="px-4 py-2 text-sm font-medium text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-md">Cancelar</button>
                        <button type="submit" class="px-4 py-2 text-sm font-medium text-white bg-primary-500 hover:bg-primary-600 rounded-md shadow-sm">Salvar</button>
                    </div>
                </form>
            </div>
        </div>
    </div>

    <!-- Confirm Delete Modal -->
    <div id="confirmDeleteModal" class="fixed inset-0 z-50 overflow-y-auto hidden">
        <div class="flex items-center justify-center min-h-screen px-4">
            <div class="fixed inset-0 bg-black bg-opacity-50 transition-opacity" id="confirmDeleteModalBackdrop"></div>
            <div class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-xl transform transition-all max-w-md w-full">
                <div class="px-6 py-4 border-b border-gray-200 dark:border-gray-700">
                    <h3 class="text-lg font-semibold text-red-600">Confirmar Exclusão</h3>
                </div>
                <div class="p-6">
                    <p id="confirmDeleteMessage" class="mb-4">Tem certeza que deseja excluir este item? Esta ação não pode ser desfeita.</p>
                    <div class="flex justify-end space-x-3">
                        <button id="cancelDelete" class="px-4 py-2 text-sm font-medium text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-md">Cancelar</button>
                        <button id="confirmDelete" class="px-4 py-2 text-sm font-medium text-white bg-red-600 hover:bg-red-700 rounded-md shadow-sm">Excluir</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Import/Export Modal -->
    <div id="dataTransferModal" class="fixed inset-0 z-50 overflow-y-auto hidden">
        <div class="flex items-center justify-center min-h-screen px-4">
            <div class="fixed inset-0 bg-black bg-opacity-50 transition-opacity" id="dataTransferModalBackdrop"></div>
            <div class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-xl transform transition-all max-w-lg w-full">
                <div class="px-6 py-4 border-b border-gray-200 dark:border-gray-700 flex justify-between items-center">
                    <h3 class="text-lg font-semibold" id="dataTransferModalTitle">Importar/Exportar Dados</h3>
                    <button id="closeDataTransferModal" class="text-gray-400 hover:text-gray-500 focus:outline-none">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
                <div id="dataTransferContent" class="p-6">
                    <div id="importContent">
                        <p class="mb-4">Importe dados de um arquivo JSON para o sistema. Isso substituirá todos os dados existentes.</p>
                        <div class="mb-4">
                            <label for="importFile" class="block text-sm font-medium mb-1">Arquivo para importar</label>
                            <input type="file" id="importFile" accept=".json" class="w-full px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md text-base dark:bg-gray-700">
                        </div>
                        <div class="flex justify-end">
                            <button id="importDataBtn" class="px-4 py-2 text-sm font-medium text-white bg-primary-500 hover:bg-primary-600 rounded-md shadow-sm">
                                <i class="fas fa-file-import mr-1"></i> Importar
                            </button>
                        </div>
                    </div>
                    <hr class="my-6 border-gray-200 dark:border-gray-700">
                    <div id="exportContent">
                        <p class="mb-4">Exporte todos os dados do sistema para um arquivo JSON.</p>
                        <div class="flex justify-end">
                            <button id="exportDataBtn" class="px-4 py-2 text-sm font-medium text-white bg-blue-500 hover:bg-blue-600 rounded-md shadow-sm">
                                <i class="fas fa-file-export mr-1"></i> Exportar
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Settings Modal -->
    <div id="settingsModal" class="fixed inset-0 z-50 overflow-y-auto hidden">
        <div class="flex items-center justify-center min-h-screen px-4">
            <div class="fixed inset-0 bg-black bg-opacity-50 transition-opacity" id="settingsModalBackdrop"></div>
            <div class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-xl transform transition-all max-w-lg w-full">
                <div class="px-6 py-4 border-b border-gray-200 dark:border-gray-700 flex justify-between items-center">
                    <h3 class="text-lg font-semibold">Configurações</h3>
                    <button id="closeSettingsModal" class="text-gray-400 hover:text-gray-500 focus:outline-none">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
                <div class="p-6">
                    <h4 class="font-medium mb-4">Preferências de exibição</h4>
                    <div class="space-y-4">
                        <div class="flex items-center justify-between">
                            <span class="text-sm">Modo escuro</span>
                            <label class="relative inline-block w-12 h-6">
                                <input type="checkbox" id="darkModeToggle" class="sr-only">
                                <span class="slider absolute cursor-pointer inset-0 bg-gray-300 dark:bg-gray-600 rounded-full transition-colors duration-200 before:absolute before:content-[''] before:h-4 before:w-4 before:left-1 before:bottom-1 before:bg-white before:rounded-full before:transition-transform before:duration-200 dark:before:transform dark:before:translate-x-6"></span>
                            </label>
                        </div>
                        <div class="flex items-center justify-between">
                            <span class="text-sm">Animações</span>
                            <label class="relative inline-block w-12 h-6">
                                <input type="checkbox" id="animationsToggle" class="sr-only" checked>
                                <span class="slider absolute cursor-pointer inset-0 bg-gray-300 dark:bg-gray-600 rounded-full transition-colors duration-200 before:absolute before:content-[''] before:h-4 before:w-4 before:left-1 before:bottom-1 before:bg-white before:rounded-full before:transition-transform before:duration-200 before:transform before:translate-x-6"></span>
                            </label>
                        </div>
                    </div>
                    
                    <hr class="my-6 border-gray-200 dark:border-gray-700">
                    
                    <h4 class="font-medium mb-4">Dados</h4>
                    <div class="space-y-4">
                        <div>
                            <button id="clearDataBtn" class="px-4 py-2 text-sm font-medium text-white bg-red-600 hover:bg-red-700 rounded-md shadow-sm w-full">
                                <i class="fas fa-trash-alt mr-1"></i> Limpar todos os dados
                            </button>
                            <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">Esta ação irá remover todos os dados do sistema e não pode ser desfeita.</p>
                        </div>
                    </div>
                    
                    <hr class="my-6 border-gray-200 dark:border-gray-700">
                    
                    <h4 class="font-medium mb-4">Sobre</h4>
                    <div class="text-sm text-gray-600 dark:text-gray-400">
                        <p>ProjetoPRO - Gerenciamento de Projetos</p>
                        <p>Versão 1.0.0</p>
                        <p class="mt-2">© 2023 - Todos os direitos reservados</p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- TEMPLATES DE VISUALIZAÇÃO -->
    <!-- Usados para injetar conteúdo dinâmico nas diferentes visualizações -->
    
    <!-- Template: Dashboard View -->
    <template id="dashboard-template">
        <div class="view-content space-y-6">
            <div class="flex flex-col md:flex-row justify-between items-start md:items-center">
                <h2 class="text-2xl font-bold mb-2 md:mb-0">Dashboard</h2>
                <div class="flex space-x-2">
                    <button id="newProjectBtn" class="text-sm bg-primary-500 hover:bg-primary-600 text-white px-4 py-2 rounded-md shadow-sm">
                        <i class="fas fa-plus mr-1"></i> Novo Projeto
                    </button>
                </div>
            </div>

            <!-- Stats Cards -->
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-4 border border-gray-200 dark:border-gray-700">
                    <div class="flex items-center">
                        <div class="flex-shrink-0 bg-blue-100 dark:bg-blue-900/30 p-3 rounded-md">
                            <i class="fas fa-folder text-blue-500 dark:text-blue-400"></i>
                        </div>
                        <div class="ml-4">
                            <p class="text-sm font-medium text-gray-500 dark:text-gray-400">Projetos Ativos</p>
                            <h3 id="activeProjectsCount" class="text-2xl font-bold">0</h3>
                        </div>
                    </div>
                </div>

                <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-4 border border-gray-200 dark:border-gray-700">
                    <div class="flex items-center">
                        <div class="flex-shrink-0 bg-green-100 dark:bg-green-900/30 p-3 rounded-md">
                            <i class="fas fa-check-circle text-green-500 dark:text-green-400"></i>
                        </div>
                        <div class="ml-4">
                            <p class="text-sm font-medium text-gray-500 dark:text-gray-400">Tarefas Concluídas</p>
                            <h3 id="completedTasksCount" class="text-2xl font-bold">0</h3>
                        </div>
                    </div>
                </div>

                <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-4 border border-gray-200 dark:border-gray-700">
                    <div class="flex items-center">
                        <div class="flex-shrink-0 bg-yellow-100 dark:bg-yellow-900/30 p-3 rounded-md">
                            <i class="fas fa-spinner text-yellow-500 dark:text-yellow-400"></i>
                        </div>
                        <div class="ml-4">
                            <p class="text-sm font-medium text-gray-500 dark:text-gray-400">Tarefas em Progresso</p>
                            <h3 id="inProgressTasksCount" class="text-2xl font-bold">0</h3>
                        </div>
                    </div>
                </div>

                <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-4 border border-gray-200 dark:border-gray-700">
                    <div class="flex items-center">
                        <div class="flex-shrink-0 bg-pink-100 dark:bg-pink-900/30 p-3 rounded-md">
                            <i class="fas fa-lightbulb text-pink-500 dark:text-pink-400"></i>
                        </div>
                        <div class="ml-4">
                            <p class="text-sm font-medium text-gray-500 dark:text-gray-400">Ideias Registradas</p>
                            <h3 id="ideasCount" class="text-2xl font-bold">0</h3>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Recent Projects -->
            <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-4 border border-gray-200 dark:border-gray-700">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg font-semibold">Projetos Recentes</h3>
                    <a href="#projects" class="text-primary-500 hover:text-primary-700 text-sm">Ver todos</a>
                </div>
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200 dark:divide-gray-700">
                        <thead class="bg-gray-50 dark:bg-gray-800">
                            <tr>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Projeto</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Progresso</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Status</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Prazo</th>
                            </tr>
                        </thead>
                        <tbody id="recentProjectsList" class="bg-white dark:bg-gray-800 divide-y divide-gray-200 dark:divide-gray-700">
                            <tr class="text-gray-500 dark:text-gray-400 text-sm">
                                <td class="px-6 py-4" colspan="4">Nenhum projeto encontrado. Crie um novo projeto para começar.</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>

            <!-- Recent Activities -->
            <div class="bg-white dark:bg-gray-800 rounded-lg shadow p-4 border border-gray-200 dark:border-gray-700">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg font-semibold">Atividades Recentes</h3>
                    <a href="#logs" class="text-primary-500 hover:text-primary-700 text-sm">Ver todas</a>
                </div>
                <div class="space-y-4">
                    <div id="recentActivitiesList">
                        <p class="text-gray-500 dark:text-gray-400 text-sm">Nenhuma atividade registrada.</p>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!-- Template: Projects View -->
    <template id="projects-template">
        <div class="view-content space-y-6">
            <div class="flex flex-col md:flex-row justify-between items-start md:items-center">
                <h2 class="text-2xl font-bold mb-2 md:mb-0">Projetos</h2>
                <div class="flex flex-wrap gap-2">
                    <div class="relative">
                        <input type="text" id="projectSearchInput" placeholder="Buscar projetos..." class="w-full md:w-64 text-sm pl-9 pr-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500 dark:bg-gray-700">
                        <i class="fas fa-search absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"></i>
                    </div>
                    <select id="projectStatusFilter" class="text-sm px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500 dark:bg-gray-700">
                        <option value="all">Todos os status</option>
                        <option value="not_started">Não iniciado</option>
                        <option value="in_progress">Em progresso</option>
                        <option value="completed">Concluído</option>
                    </select>
                    <button id="newProjectBtnAlt" class="text-sm bg-primary-500 hover:bg-primary-600 text-white px-4 py-2 rounded-md shadow-sm">
                        <i class="fas fa-plus mr-1"></i> Novo Projeto
                    </button>
                </div>
            </div>

            <!-- Projects Grid -->
            <div id="projectsGrid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                <!-- Projects will be loaded dynamically -->
                <p class="text-gray-500 dark:text-gray-400 text-sm col-span-full">Carregando projetos...</p>
            </div>
        </div>
    </template>

    <!-- Template: Tasks View -->
    <template id="tasks-template">
        <div class="view-content space-y-6">
            <div class="flex flex-col md:flex-row justify-between items-start md:items-center">
                <h2 class="text-2xl font-bold mb-2 md:mb-0">Tarefas</h2>
                <div class="flex flex-wrap gap-2">
                    <div class="relative">
                        <input type="text" id="taskSearchInput" placeholder="Buscar tarefas..." class="w-full md:w-64 text-sm pl-9 pr-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500 dark:bg-gray-700">
                        <i class="fas fa-search absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"></i>
                    </div>
                    <select id="taskProjectFilter" class="text-sm px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500 dark:bg-gray-700">
                        <option value="all">Todos os projetos</option>
                        <!-- Project options will be loaded dynamically -->
                    </select>
                    <select id="taskStatusFilter" class="text-sm px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500 dark:bg-gray-700">
                        <option value="all">Todos os status</option>
                        <option value="todo">A Fazer</option>
                        <option value="in_progress">Em Progresso</option>
                        <option value="completed">Concluídas</option>
                    </select>
                    <button id="newTaskBtn" class="text-sm bg-primary-500 hover:bg-primary-600 text-white px-4 py-2 rounded-md shadow-sm">
                        <i class="fas fa-plus mr-1"></i> Nova Tarefa
                    </button>
                </div>
            </div>

            <!-- Tasks Table -->
            <div class="bg-white dark:bg-gray-800 rounded-lg shadow overflow-hidden border border-gray-200 dark:border-gray-700">
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200 dark:divide-gray-700">
                        <thead class="bg-gray-50 dark:bg-gray-700">
                            <tr>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Nome</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Projeto</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Status</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Prioridade</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Prazo</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Ações</th>
                            </tr>
                        </thead>
                        <tbody id="tasksTableBody" class="bg-white dark:bg-gray-800 divide-y divide-gray-200 dark:divide-gray-700">
                            <!-- Tasks will be loaded dynamically -->
                            <tr>
                                <td colspan="6" class="px-6 py-4 text-center text-sm text-gray-500 dark:text-gray-400">Carregando tarefas...</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </template>

    <!-- Template: Kanban View -->
    <template id="kanban-template">
        <div class="view-content space-y-6">
            <div class="flex flex-col md:flex-row justify-between items-start md:items-center">
                <h2 class="text-2xl font-bold mb-2 md:mb-0">Kanban</h2>
                <div class="flex flex-wrap gap-2">
                    <select id="kanbanProjectFilter" class="text-sm px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500 dark:bg-gray-700">
                        <option value="all">Todos os projetos</option>
                        <!-- Project options will be loaded dynamically -->
                    </select>
                    <button id="newTaskBtnAlt" class="text-sm bg-primary-500 hover:bg-primary-600 text-white px-4 py-2 rounded-md shadow-sm">
                        <i class="fas fa-plus mr-1"></i> Nova Tarefa
                    </button>
                </div>
            </div>

            <!-- Kanban Board -->
            <div class="flex flex-col lg:flex-row gap-4 overflow-x-auto pb-4">
                <!-- To Do Column -->
                <div class="flex-1 min-w-[300px]">
                    <div class="bg-gray-100 dark:bg-gray-700 p-3 rounded-t-lg">
                        <h3 class="font-medium flex items-center">
                            <span class="w-3 h-3 bg-gray-400 rounded-full mr-2"></span>
                            A Fazer
                            <span id="todoCount" class="ml-2 bg-gray-200 dark:bg-gray-600 text-gray-700 dark:text-gray-300 text-xs rounded-full px-2 py-0.5">0</span>
                        </h3>
                    </div>
                    <div id="todoColumn" class="bg-white dark:bg-gray-800 p-2 min-h-[300px] rounded-b-lg border border-gray-200 dark:border-gray-700 drop-zone" data-status="todo">
                        <!-- Tasks will be loaded dynamically -->
                    </div>
                </div>

                <!-- In Progress Column -->
                <div class="flex-1 min-w-[300px]">
                    <div class="bg-blue-100 dark:bg-blue-900/30 p-3 rounded-t-lg">
                        <h3 class="font-medium flex items-center">
                            <span class="w-3 h-3 bg-blue-500 rounded-full mr-2"></span>
                            Em Progresso
                            <span id="inProgressCount" class="ml-2 bg-blue-200 dark:bg-blue-800/50 text-blue-700 dark:text-blue-300 text-xs rounded-full px-2 py-0.5">0</span>
                        </h3>
                    </div>
                    <div id="inProgressColumn" class="bg-white dark:bg-gray-800 p-2 min-h-[300px] rounded-b-lg border border-gray-200 dark:border-gray-700 drop-zone" data-status="in_progress">
                        <!-- Tasks will be loaded dynamically -->
                    </div>
                </div>

                <!-- Completed Column -->
                <div class="flex-1 min-w-[300px]">
                    <div class="bg-green-100 dark:bg-green-900/30 p-3 rounded-t-lg">
                        <h3 class="font-medium flex items-center">
                            <span class="w-3 h-3 bg-green-500 rounded-full mr-2"></span>
                            Concluídas
                            <span id="completedCount" class="ml-2 bg-green-200 dark:bg-green-800/50 text-green-700 dark:text-green-300 text-xs rounded-full px-2 py-0.5">0</span>
                        </h3>
                    </div>
                    <div id="completedColumn" class="bg-white dark:bg-gray-800 p-2 min-h-[300px] rounded-b-lg border border-gray-200 dark:border-gray-700 drop-zone" data-status="completed">
                        <!-- Tasks will be loaded dynamically -->
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!-- Template: Timeline View -->
    <template id="timeline-template">
        <div class="view-content space-y-6">
            <div class="flex flex-col md:flex-row justify-between items-start md:items-center">
                <h2 class="text-2xl font-bold mb-2 md:mb-0">Cronograma</h2>
                <div class="flex flex-wrap gap-2">
                    <select id="timelineProjectFilter" class="text-sm px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500 dark:bg-gray-700">
                        <option value="all">Todos os projetos</option>
                        <!-- Project options will be loaded dynamically -->
                    </select>
                    <div class="flex">
                        <button id="timelineView" class="text-sm px-3 py-2 rounded-l-md bg-primary-500 text-white">
                            <i class="fas fa-stream"></i>
                        </button>
                        <button id="calendarView" class="text-sm px-3 py-2 rounded-r-md bg-gray-200 dark:bg-gray-700 text-gray-700 dark:text-gray-300">
                            <i class="fas fa-calendar-alt"></i>
                        </button>
                    </div>
                </div>
            </div>

            <!-- Timeline View Container -->
            <div id="timelineContainer" class="bg-white dark:bg-gray-800 p-4 rounded-lg shadow border border-gray-200 dark:border-gray-700 min-h-[400px]">
                <!-- Timeline will be rendered here dynamically -->
            </div>

            <!-- Calendar View Container (hidden by default) -->
            <div id="calendarContainer" class="bg-white dark:bg-gray-800 p-4 rounded-lg shadow border border-gray-200 dark:border-gray-700 hidden">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg font-semibold" id="calendarMonthYear">Mês Ano</h3>
                    <div class="flex space-x-2">
                        <button id="prevMonth" class="p-1 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700">
                            <i class="fas fa-chevron-left"></i>
                        </button>
                        <button id="nextMonth" class="p-1 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700">
                            <i class="fas fa-chevron-right"></i>
                        </button>
                    </div>
                </div>
                
                <!-- Calendar Grid -->
                <div id="calendarGrid" class="grid grid-cols-7 gap-1">
                    <!-- Calendar header -->
                    <div class="text-center p-2 font-medium">Dom</div>
                    <div class="text-center p-2 font-medium">Seg</div>
                    <div class="text-center p-2 font-medium">Ter</div>
                    <div class="text-center p-2 font-medium">Qua</div>
                    <div class="text-center p-2 font-medium">Qui</div>
                    <div class="text-center p-2 font-medium">Sex</div>
                    <div class="text-center p-2 font-medium">Sáb</div>
                    
                    <!-- Calendar days will be generated dynamically -->
                </div>
                
                <!-- Selected Day Events -->
                <div id="dayEventsContainer" class="mt-6 hidden">
                    <h3 id="dayEventsTitle" class="text-lg font-semibold mb-3">Eventos do dia</h3>
                    <div id="dayEventsList" class="space-y-3">
                        <!-- Events will be loaded dynamically -->
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!-- Template: Logs View -->
    <template id="logs-template">
        <div class="view-content space-y-6">
            <div class="flex justify-between items-center">
                <h2 class="text-2xl font-bold">Registros de Atividades</h2>
                <div class="flex flex-wrap gap-2">
                    <select id="logTypeFilter" class="text-sm px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500 dark:bg-gray-700">
                        <option value="all">Todos os tipos</option>
                        <option value="project_created">Projeto criado</option>
                        <option value="project_updated">Projeto atualizado</option>
                        <option value="project_deleted">Projeto excluído</option>
                        <option value="task_created">Tarefa criada</option>
                        <option value="task_updated">Tarefa atualizada</option>
                        <option value="task_deleted">Tarefa excluída</option>
                        <option value="task_completed">Tarefa concluída</option>
                        <option value="idea_created">Ideia registrada</option>
                    </select>
                </div>
            </div>

            <!-- Logs Timeline -->
            <div class="relative pl-8 space-y-8 before:content-[''] before:absolute before:top-2 before:bottom-0 before:w-0.5 before:left-0 before:bg-gray-200 dark:before:bg-gray-700 ml-6">
                <div id="logsTimeline">
                    <!-- Logs will be loaded dynamically -->
                    <p class="text-gray-500 dark:text-gray-400 -ml-6">Carregando registros...</p>
                </div>
            </div>
        </div>
    </template>

    <!-- Template: Ideas View -->
    <template id="ideas-template">
        <div class="view-content space-y-6">
            <div class="flex flex-col md:flex-row justify-between items-start md:items-center">
                <h2 class="text-2xl font-bold mb-2 md:mb-0">Ideias</h2>
                <div class="flex flex-wrap gap-2">
                    <div class="relative">
                        <input type="text" id="ideaSearchInput" placeholder="Buscar ideias..." class="w-full md:w-64 text-sm pl-9 pr-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500 dark:bg-gray-700">
                        <i class="fas fa-search absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"></i>
                    </div>
                    <select id="ideaCategoryFilter" class="text-sm px-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500 dark:bg-gray-700">
                        <option value="all">Todas as categorias</option>
                        <option value="feature">Nova funcionalidade</option>
                        <option value="improvement">Melhoria</option>
                        <option value="project">Projeto</option>
                        <option value="process">Processo</option>
                        <option value="other">Outra</option>
                    </select>
                    <button id="newIdeaBtn" class="text-sm bg-primary-500 hover:bg-primary-600 text-white px-4 py-2 rounded-md shadow-sm">
                        <i class="fas fa-plus mr-1"></i> Nova Ideia
                    </button>
                </div>
            </div>

            <!-- Ideas Grid -->
            <div id="ideasGrid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                <!-- Ideas will be loaded dynamically -->
                <p class="text-gray-500 dark:text-gray-400 text-sm col-span-full">Carregando ideias...</p>
            </div>
        </div>
    </template>

    <!-- Main JavaScript -->
    <script>
        // Variáveis globais
        let store = {
            projects: [],
            tasks: [],
            logs: [],
            ideas: []
        };
        
        // Identificador para o item atual em exclusão/edição
        let currentItemForDeletion = null;
        let currentView = 'dashboard';
        let currentTimePeriod = new Date();

        // Função para carregar dados do localStorage
        function loadStore() {
            try {
                const data = localStorage.getItem('projetoPRO_data');
                if (data) {
                    const parsedData = JSON.parse(data);
                    store = parsedData;
                    console.log('Dados carregados com sucesso:', store);
                    return true;
                }
                return false;
            } catch (error) {
                console.error('Erro ao carregar dados:', error);
                return false;
            }
        }

        // Função para salvar dados no localStorage
        function saveStore() {
            try {
                localStorage.setItem('projetoPRO_data', JSON.stringify(store));
                console.log('Dados salvos com sucesso');
                return true;
            } catch (error) {
                console.error('Erro ao salvar dados:', error);
                return false;
            }
        }

        // Função para inicializar dados de exemplo (se necessário)
        function initializeExampleData() {
            // Adicionar projetos de exemplo
            const projects = [
                {
                    id: 'proj_' + Date.now(),
                    name: 'Redesign do Website',
                    description: 'Modernização completa do website corporativo para melhorar a experiência do usuário',
                    startDate: new Date(Date.now() - 15 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    dueDate: new Date(Date.now() + 45 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    priority: 'high',
                    status: 'in_progress',
                    createdAt: new Date(Date.now() - 15 * 24 * 60 * 60 * 1000).toISOString(),
                    why: 'Para melhorar a conversão e modernizar a imagem da empresa',
                    who: 'Equipe de Marketing e Desenvolvimento Web',
                    where: 'Site principal da empresa',
                    how: 'Utilizando design responsivo e tecnologias modernas',
                    howMuch: 'R$ 25.000,00'
                },
                {
                    id: 'proj_' + (Date.now() + 1),
                    name: 'Desenvolvimento do App Mobile',
                    description: 'Criar um aplicativo móvel para clientes com funcionalidades principais do sistema',
                    startDate: new Date(Date.now() - 5 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    dueDate: new Date(Date.now() + 60 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    priority: 'medium',
                    status: 'in_progress',
                    createdAt: new Date(Date.now() - 5 * 24 * 60 * 60 * 1000).toISOString(),
                    why: 'Para expandir o acesso dos clientes ao sistema em dispositivos móveis',
                    who: 'Equipe de Desenvolvimento Mobile',
                    where: 'Aplicativo para Android e iOS',
                    how: 'Utilizando React Native para desenvolvimento cross-platform',
                    howMuch: 'R$ 60.000,00'
                }
            ];
            
            store.projects = projects;
            
            // Adicionar tarefas de exemplo para cada projeto
            const tasks = [
                {
                    id: 'task_' + Date.now(),
                    name: 'Análise de requisitos do website',
                    description: 'Levantar todos os requisitos e funcionalidades necessárias para o novo website',
                    projectId: projects[0].id,
                    status: 'completed',
                    priority: 'high',
                    dueDate: new Date(Date.now() - 10 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    createdAt: new Date(Date.now() - 15 * 24 * 60 * 60 * 1000).toISOString(),
                    who: 'Ana Silva',
                    how: 'Reuniões com stakeholders e análise do site atual'
                },
                {
                    id: 'task_' + (Date.now() + 1),
                    name: 'Design de wireframes',
                    description: 'Criar wireframes para todas as páginas principais do site',
                    projectId: projects[0].id,
                    status: 'completed',
                    priority: 'medium',
                    dueDate: new Date(Date.now() - 5 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    createdAt: new Date(Date.now() - 12 * 24 * 60 * 60 * 1000).toISOString(),
                    who: 'Carlos Mendes',
                    how: 'Utilizando Figma para criação dos wireframes'
                },
                {
                    id: 'task_' + (Date.now() + 2),
                    name: 'Desenvolvimento Frontend',
                    description: 'Implementar o design em HTML, CSS e JavaScript',
                    projectId: projects[0].id,
                    status: 'in_progress',
                    priority: 'medium',
                    dueDate: new Date(Date.now() + 10 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    createdAt: new Date(Date.now() - 5 * 24 * 60 * 60 * 1000).toISOString(),
                    who: 'Pedro Alves',
                    how: 'Utilizando React e Tailwind CSS'
                },
                {
                    id: 'task_' + (Date.now() + 3),
                    name: 'Desenvolvimento Backend',
                    description: 'Implementar API e funcionalidades de backend',
                    projectId: projects[0].id,
                    status: 'todo',
                    priority: 'high',
                    dueDate: new Date(Date.now() + 20 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    createdAt: new Date(Date.now() - 5 * 24 * 60 * 60 * 1000).toISOString(),
                    who: 'Marcela Souza',
                    how: 'Utilizando Node.js e MongoDB'
                },
                {
                    id: 'task_' + (Date.now() + 4),
                    name: 'Pesquisa de mercado para apps similares',
                    description: 'Analisar aplicativos concorrentes e identificar funcionalidades diferenciais',
                    projectId: projects[1].id,
                    status: 'completed',
                    priority: 'medium',
                    dueDate: new Date(Date.now() - 2 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    createdAt: new Date(Date.now() - 5 * 24 * 60 * 60 * 1000).toISOString(),
                    who: 'Fernanda Lima',
                    how: 'Análise de apps concorrentes e pesquisa com usuários'
                },
                {
                    id: 'task_' + (Date.now() + 5),
                    name: 'Prototipagem do app',
                    description: 'Criar protótipos interativos para as principais telas do aplicativo',
                    projectId: projects[1].id,
                    status: 'in_progress',
                    priority: 'high',
                    dueDate: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    createdAt: new Date(Date.now() - 3 * 24 * 60 * 60 * 1000).toISOString(),
                    who: 'Ricardo Gomes',
                    how: 'Utilizando Figma para criação de protótipos interativos'
                },
                {
                    id: 'task_' + (Date.now() + 6),
                    name: 'Desenvolvimento da estrutura base do app',
                    description: 'Criar a estrutura básica do aplicativo e configurar o ambiente',
                    projectId: projects[1].id,
                    status: 'todo',
                    priority: 'high',
                    dueDate: new Date(Date.now() + 15 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                    createdAt: new Date(Date.now() - 3 * 24 * 60 * 60 * 1000).toISOString(),
                    who: 'Lucas Oliveira',
                    how: 'Configuração do React Native e estrutura de pastas'
                }
            ];
            
            store.tasks = tasks;
            
            // Adicionar ideias de exemplo
            const ideas = [
                {
                    id: 'idea_' + Date.now(),
                    title: 'Sistema de gamificação para usuários',
                    description: 'Implementar sistema de pontos, conquistas e rankings para aumentar o engajamento dos usuários com a plataforma.',
                    category: 'feature',
                    tags: ['gamificação', 'engajamento', 'experiência do usuário'],
                    createdAt: new Date(Date.now() - 8 * 24 * 60 * 60 * 1000).toISOString()
                },
                {
                    id: 'idea_' + (Date.now() + 1),
                    title: 'Aplicativo para wearables',
                    description: 'Desenvolver uma versão simplificada do app para smartwatches e outros dispositivos vestíveis.',
                    category: 'project',
                    tags: ['wearables', 'inovação', 'expansão'],
                    createdAt: new Date(Date.now() - 3 * 24 * 60 * 60 * 1000).toISOString()
                }
            ];
            
            store.ideas = ideas;
            
            // Registrar logs de exemplo
            const logs = [
                {
                    id: 'log_' + Date.now(),
                    type: 'project_created',
                    projectId: projects[0].id,
                    message: `Projeto "${projects[0].name}" criado`,
                    createdAt: projects[0].createdAt
                },
                {
                    id: 'log_' + (Date.now() + 1),
                    type: 'project_created',
                    projectId: projects[1].id,
                    message: `Projeto "${projects[1].name}" criado`,
                    createdAt: projects[1].createdAt
                }
            ];
            
            // Adicionar logs para cada tarefa
            tasks.forEach(task => {
                logs.push({
                    id: 'log_' + (Date.now() + logs.length),
                    type: 'task_created',
                    taskId: task.id,
                    projectId: task.projectId,
                    message: `Tarefa "${task.name}" criada no projeto "${getProjectName(task.projectId)}"`,
                    createdAt: task.createdAt
                });
                
                if (task.status === 'completed') {
                    logs.push({
                        id: 'log_' + (Date.now() + logs.length),
                        type: 'task_completed',
                        taskId: task.id,
                        projectId: task.projectId,
                        message: `Tarefa "${task.name}" concluída no projeto "${getProjectName(task.projectId)}"`,
                        createdAt: new Date(new Date(task.dueDate).getTime() - 24 * 60 * 60 * 1000).toISOString()
                    });
                }
            });
            
            // Adicionar logs para cada ideia
            ideas.forEach(idea => {
                logs.push({
                    id: 'log_' + (Date.now() + logs.length),
                    type: 'idea_created',
                    ideaId: idea.id,
                    message: `Ideia "${idea.title}" registrada`,
                    createdAt: idea.createdAt
                });
            });
            
            store.logs = logs;
            
            saveStore();
            console.log('Dados de exemplo inicializados');
            
            // Função auxiliar para obter o nome do projeto
            function getProjectName(projectId) {
                const project = projects.find(p => p.id === projectId);
                return project ? project.name : 'Desconhecido';
            }
        }

        // Função para exibir notificações
        function showToast(message, type = 'info') {
            const toastContainer = document.getElementById('toastContainer');
            if (!toastContainer) return;
            
            const toast = document.createElement('div');
            toast.className = `toast shadow-md rounded-lg p-4 mb-2`;
            
            // Definir cor com base no tipo
            let bgColor, iconClass;
            switch (type) {
                case 'success':
                    bgColor = 'bg-green-500';
                    iconClass = 'fa-check-circle';
                    break;
                case 'error':
                    bgColor = 'bg-red-500';
                    iconClass = 'fa-exclamation-circle';
                    break;
                case 'warning':
                    bgColor = 'bg-yellow-500';
                    iconClass = 'fa-exclamation-triangle';
                    break;
                default:
                    bgColor = 'bg-blue-500';
                    iconClass = 'fa-info-circle';
            }
            
            toast.className += ` ${bgColor} text-white`;
            
            toast.innerHTML = `
                <div class="flex items-center">
                    <i class="fas ${iconClass} mr-2"></i>
                    <span>${message}</span>
                </div>
            `;
            
            toastContainer.appendChild(toast);
            
            // Remover após 3 segundos
            setTimeout(() => {
                if (toast.parentNode) {
                    toast.parentNode.removeChild(toast);
                }
            }, 3000);
        }

        // Função para atualizar estatísticas
        function updateStats() {
            // Atualizar contadores
            document.getElementById('activeProjectsCount').textContent = store.projects.length;
            document.getElementById('completedTasksCount').textContent = store.tasks.filter(t => t.status === 'completed').length;
            document.getElementById('inProgressTasksCount').textContent = store.tasks.filter(t => t.status === 'in_progress').length;
            document.getElementById('ideasCount').textContent = store.ideas.length;
        }

        // Atualizar o indicador de armazenamento
        function updateStorageIndicator() {
            try {
                const storageIndicator = document.getElementById('storageIndicator');
                const storageUsed = document.getElementById('storageUsed');
                
                if (!storageIndicator || !storageUsed) return;
                
                // Calcular o tamanho aproximado dos dados em KB
                const dataSize = JSON.stringify(store).length / 1024;
                const formattedSize = dataSize.toFixed(2);
                
                // Atualizar texto
                storageUsed.textContent = `${formattedSize} KB`;
                
                // Atualizar barra de progresso (máximo arbitrário de 5 MB para exemplo)
                const maxStorage = 5 * 1024; // 5 MB em KB
                const percentUsed = Math.min((dataSize / maxStorage) * 100, 100);
                storageIndicator.style.width = `${percentUsed}%`;
                
            } catch (error) {
                console.error('Erro ao atualizar indicador de armazenamento:', error);
            }
        }

        // Função para formatar data
        function formatDate(dateString) {
            if (!dateString) return 'Não definido';
            const date = new Date(dateString);
            return date.toLocaleDateString('pt-BR');
        }

        // Funções de gerenciamento de projetos
        function createProject(projectData) {
            const newProject = {
                id: 'proj_' + Date.now(),
                createdAt: new Date().toISOString(),
                status: 'in_progress',
                ...projectData
            };
            
            store.projects.push(newProject);
            
            // Registrar log
            addLog({
                type: 'project_created',
                projectId: newProject.id,
                message: `Projeto "${newProject.name}" criado`
            });
            
            saveStore();
            
            return newProject;
        }

        function updateProject(projectId, projectData) {
            const index = store.projects.findIndex(p => p.id === projectId);
            if (index === -1) return false;
            
            const updatedProject = { ...store.projects[index], ...projectData, updatedAt: new Date().toISOString() };
            store.projects[index] = updatedProject;
            
            // Registrar log
            addLog({
                type: 'project_updated',
                projectId: updatedProject.id,
                message: `Projeto "${updatedProject.name}" atualizado`
            });
            
            saveStore();
            
            return updatedProject;
        }

        function deleteProject(projectId) {
            const projectIndex = store.projects.findIndex(p => p.id === projectId);
            if (projectIndex === -1) return false;
            
            const projectName = store.projects[projectIndex].name;
            
            // Remover o projeto
            store.projects.splice(projectIndex, 1);
            
            // Remover todas as tarefas relacionadas
            const relatedTasks = store.tasks.filter(t => t.projectId === projectId);
            store.tasks = store.tasks.filter(t => t.projectId !== projectId);
            
            // Registrar log
            addLog({
                type: 'project_deleted',
                message: `Projeto "${projectName}" excluído com ${relatedTasks.length} tarefas`
            });
            
            saveStore();
            
            return true;
        }

        // Funções de gerenciamento de tarefas
        function createTask(taskData) {
            const newTask = {
                id: 'task_' + Date.now(),
                createdAt: new Date().toISOString(),
                status: 'todo',
                ...taskData
            };
            
            store.tasks.push(newTask);
            
            // Registrar log
            const project = store.projects.find(p => p.id === newTask.projectId);
            addLog({
                type: 'task_created',
                taskId: newTask.id,
                projectId: newTask.projectId,
                message: `Tarefa "${newTask.name}" criada no projeto "${project ? project.name : 'Desconhecido'}"`
            });
            
            saveStore();
            
            return newTask;
        }

        function updateTask(taskId, taskData) {
            const index = store.tasks.findIndex(t => t.id === taskId);
            if (index === -1) return false;
            
            const oldStatus = store.tasks[index].status;
            const updatedTask = { ...store.tasks[index], ...taskData, updatedAt: new Date().toISOString() };
            store.tasks[index] = updatedTask;
            
            // Registrar log
            const project = store.projects.find(p => p.id === updatedTask.projectId);
            addLog({
                type: 'task_updated',
                taskId: updatedTask.id,
                projectId: updatedTask.projectId,
                message: `Tarefa "${updatedTask.name}" atualizada no projeto "${project ? project.name : 'Desconhecido'}"`
            });
            
            // Registrar conclusão de tarefa
            if (oldStatus !== 'completed' && updatedTask.status === 'completed') {
                addLog({
                    type: 'task_completed',
                    taskId: updatedTask.id,
                    projectId: updatedTask.projectId,
                    message: `Tarefa "${updatedTask.name}" concluída no projeto "${project ? project.name : 'Desconhecido'}"`
                });
            }
            
            saveStore();
            
            return updatedTask;
        }

        function deleteTask(taskId) {
            const taskIndex = store.tasks.findIndex(t => t.id === taskId);
            if (taskIndex === -1) return false;
            
            const task = store.tasks[taskIndex];
            const projectName = store.projects.find(p => p.id === task.projectId)?.name || 'Desconhecido';
            
            // Remover a tarefa
            store.tasks.splice(taskIndex, 1);
            
            // Registrar log
            addLog({
                type: 'task_deleted',
                projectId: task.projectId,
                message: `Tarefa "${task.name}" excluída do projeto "${projectName}"`
            });
            
            saveStore();
            
            return true;
        }

        // Funções de gerenciamento de ideias
        function createIdea(ideaData) {
            const newIdea = {
                id: 'idea_' + Date.now(),
                createdAt: new Date().toISOString(),
                ...ideaData
            };
            
            store.ideas.push(newIdea);
            
            // Registrar log
            addLog({
                type: 'idea_created',
                ideaId: newIdea.id,
                message: `Ideia "${newIdea.title}" registrada`
            });
            
            saveStore();
            
            return newIdea;
        }

        function updateIdea(ideaId, ideaData) {
            const index = store.ideas.findIndex(i => i.id === ideaId);
            if (index === -1) return false;
            
            const updatedIdea = { ...store.ideas[index], ...ideaData, updatedAt: new Date().toISOString() };
            store.ideas[index] = updatedIdea;
            
            // Registrar log
            addLog({
                type: 'idea_updated',
                ideaId: updatedIdea.id,
                message: `Ideia "${updatedIdea.title}" atualizada`
            });
            
            saveStore();
            
            return updatedIdea;
        }

        function deleteIdea(ideaId) {
            const ideaIndex = store.ideas.findIndex(i => i.id === ideaId);
            if (ideaIndex === -1) return false;
            
            const ideaTitle = store.ideas[ideaIndex].title;
            
            // Remover a ideia
            store.ideas.splice(ideaIndex, 1);
            
            // Registrar log
            addLog({
                type: 'idea_deleted',
                message: `Ideia "${ideaTitle}" excluída`
            });
            
            saveStore();
            
            return true;
        }

        // Função para adicionar log
        function addLog(logData) {
            const newLog = {
                id: 'log_' + Date.now(),
                createdAt: new Date().toISOString(),
                ...logData
            };
            
            store.logs.push(newLog);
            saveStore();
            
            return newLog;
        }

        // FUNÇÕES DE RENDERIZAÇÃO DE VISUALIZAÇÃO

        // Renderizar dashboard
        function renderDashboard() {
            const template = document.getElementById('dashboard-template');
            document.getElementById('pageContent').innerHTML = template.innerHTML;
            
            updateStats();
            
            // Renderizar projetos recentes
            const recentProjectsList = document.getElementById('recentProjectsList');
            
            if (store.projects.length > 0) {
                // Ordenar por data de criação (mais recentes primeiro)
                const sortedProjects = [...store.projects].sort((a, b) => {
                    return new Date(b.createdAt) - new Date(a.createdAt);
                }).slice(0, 5); // Mostrar apenas os 5 mais recentes
                
                recentProjectsList.innerHTML = sortedProjects.map(project => {
                    // Calcular progresso
                    const projectTasks = store.tasks.filter(t => t.projectId === project.id);
                    const totalTasks = projectTasks.length;
                    const completedTasks = projectTasks.filter(t => t.status === 'completed').length;
                    const progress = totalTasks > 0 ? Math.round((completedTasks / totalTasks) * 100) : 0;
                    
                    // Status baseado no progresso
                    const statusClass = 
                        progress === 100 ? 'bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-300' :
                        progress >= 70 ? 'bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-300' :
                        progress >= 30 ? 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/30 dark:text-yellow-300' :
                        'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
                    
                    const statusText = 
                        progress === 100 ? 'Concluído' :
                        progress >= 70 ? 'Avançado' :
                        progress >= 30 ? 'Em progresso' :
                        'Iniciando';
                    
                    // Formatar data
                    const dueDate = project.dueDate ? formatDate(project.dueDate) : 'Não definido';
                    
                    return `
                        <tr class="hover:bg-gray-50 dark:hover:bg-gray-700/40 cursor-pointer project-row" data-id="${project.id}">
                            <td class="px-6 py-4">
                                <div class="flex items-center">
                                    <span class="font-medium">${project.name}</span>
                                </div>
                            </td>
                            <td class="px-6 py-4">
                                <div class="w-full bg-gray-200 dark:bg-gray-700 rounded-full h-2.5">
                                    <div class="bg-primary-500 h-2.5 rounded-full" style="width: ${progress}%"></div>
                                </div>
                                <span class="text-xs text-gray-500 dark:text-gray-400">${progress}% (${completedTasks}/${totalTasks})</span>
                            </td>
                            <td class="px-6 py-4">
                                <span class="px-2 py-1 rounded-full text-xs ${statusClass}">
                                    ${statusText}
                                </span>
                            </td>
                            <td class="px-6 py-4 text-sm">
                                ${dueDate}
                            </td>
                        </tr>
                    `;
                }).join('');
                
                // Adicionar eventos aos itens da tabela
                document.querySelectorAll('.project-row').forEach(row => {
                    row.addEventListener('click', function() {
                        const projectId = this.getAttribute('data-id');
                        openProjectDetail(projectId);
                    });
                });
            } else {
                recentProjectsList.innerHTML = `
                    <tr class="text-gray-500 dark:text-gray-400 text-sm">
                        <td class="px-6 py-4" colspan="4">Nenhum projeto encontrado. Crie um novo projeto para começar.</td>
                    </tr>
                `;
            }
            
            // Renderizar atividades recentes
            const recentActivitiesList = document.getElementById('recentActivitiesList');
            
            if (store.logs.length > 0) {
                // Ordenar por data (mais recentes primeiro)
                const sortedLogs = [...store.logs].sort((a, b) => {
                    return new Date(b.createdAt) - new Date(a.createdAt);
                }).slice(0, 5); // Mostrar apenas os 5 mais recentes
                
                recentActivitiesList.innerHTML = sortedLogs.map(log => {
                    // Formatar data
                    const formattedDate = new Date(log.createdAt).toLocaleString('pt-BR');
                    
                    // Ícone baseado no tipo
                    let icon, color;
                    switch (log.type) {
                        case 'project_created':
                            icon = 'folder-plus';
                            color = 'text-blue-500 dark:text-blue-400';
                            break;
                        case 'task_completed':
                            icon = 'check-circle';
                            color = 'text-green-500 dark:text-green-400';
                            break;
                        case 'task_created':
                            icon = 'tasks';
                            color = 'text-yellow-500 dark:text-yellow-400';
                            break;
                        case 'idea_created':
                            icon = 'lightbulb';
                            color = 'text-pink-500 dark:text-pink-400';
                            break;
                        default:
                            icon = 'history';
                            color = 'text-gray-500 dark:text-gray-400';
                    }
                    
                    return `
                        <div class="flex space-x-4 pb-4">
                            <div class="flex-shrink-0">
                                <div class="w-8 h-8 rounded-full bg-gray-100 dark:bg-gray-700 flex items-center justify-center">
                                    <i class="fas fa-${icon} ${color}"></i>
                                </div>
                            </div>
                            <div class="flex-1">
                                <p class="text-sm">${log.message}</p>
                                <p class="text-xs text-gray-500 dark:text-gray-400">${formattedDate}</p>
                            </div>
                        </div>
                    `;
                }).join('');
            } else {
                recentActivitiesList.innerHTML = `<p class="text-gray-500 dark:text-gray-400 text-sm">Nenhuma atividade registrada.</p>`;
            }
            
            // Adicionar evento ao botão novo projeto
            document.getElementById('newProjectBtn').addEventListener('click', function() {
                openNewProjectModal();
            });
        }

        // Renderizar projetos
        function renderProjects() {
            const template = document.getElementById('projects-template');
            document.getElementById('pageContent').innerHTML = template.innerHTML;
            
            const projectsGrid = document.getElementById('projectsGrid');
            const searchInput = document.getElementById('projectSearchInput');
            const statusFilter = document.getElementById('projectStatusFilter');
            
            // Adicionar eventos de filtro
            searchInput.addEventListener('input', filterProjects);
            statusFilter.addEventListener('change', filterProjects);
            
            // Adicionar evento ao botão novo projeto
            document.getElementById('newProjectBtnAlt').addEventListener('click', function() {
                openNewProjectModal();
            });
            
            // Carregar projetos
            filterProjects();
            
            function filterProjects() {
                const searchTerm = searchInput.value.toLowerCase();
                const statusValue = statusFilter.value;
                
                let filteredProjects = [...store.projects];
                
                // Aplicar filtro de busca
                if (searchTerm) {
                    filteredProjects = filteredProjects.filter(project => 
                        project.name.toLowerCase().includes(searchTerm) || 
                        (project.description && project.description.toLowerCase().includes(searchTerm))
                    );
                }
                
                // Aplicar filtro de status
                if (statusValue !== 'all') {
                    filteredProjects = filteredProjects.filter(project => project.status === statusValue);
                }
                
                // Ordenar por data de criação (mais recentes primeiro)
                filteredProjects.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
                
                if (filteredProjects.length > 0) {
                    projectsGrid.innerHTML = filteredProjects.map(project => {
                        // Calcular progresso
                        const projectTasks = store.tasks.filter(t => t.projectId === project.id);
                        const totalTasks = projectTasks.length;
                        const completedTasks = projectTasks.filter(t => t.status === 'completed').length;
                        const progress = totalTasks > 0 ? Math.round((completedTasks / totalTasks) * 100) : 0;
                        
                        // Status baseado no progresso
                        let statusBadge, statusColor;
                        if (progress === 100) {
                            statusBadge = 'Concluído';
                            statusColor = 'bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-300';
                        } else if (progress >= 70) {
                            statusBadge = 'Avançado';
                            statusColor = 'bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-300';
                        } else if (progress >= 30) {
                            statusBadge = 'Em progresso';
                            statusColor = 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/30 dark:text-yellow-300';
                        } else {
                            statusBadge = 'Iniciando';
                            statusColor = 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
                        }
                        
                        // Prioridade
                        let priorityColor;
                        switch (project.priority) {
                            case 'high':
                                priorityColor = 'text-red-500 dark:text-red-400';
                                break;
                            case 'medium':
                                priorityColor = 'text-yellow-500 dark:text-yellow-400';
                                break;
                            default:
                                priorityColor = 'text-gray-500 dark:text-gray-400';
                        }
                        
                        // Datas formatadas
                        const startDate = project.startDate ? formatDate(project.startDate) : 'Não definido';
                        const dueDate = project.dueDate ? formatDate(project.dueDate) : 'Não definido';
                        
                        return `
                            <div class="bg-white dark:bg-gray-800 rounded-lg shadow overflow-hidden border border-gray-200 dark:border-gray-700 hover:shadow-md transition-shadow">
                                <div class="p-4">
                                    <div class="flex justify-between items-start">
                                        <h3 class="text-lg font-semibold flex-1">${project.name}</h3>
                                        <span class="px-2 py-1 rounded-full text-xs ${statusColor}">
                                            ${statusBadge}
                                        </span>
                                    </div>
                                    <p class="text-sm text-gray-600 dark:text-gray-400 mt-1 line-clamp-2">${project.description || 'Sem descrição'}</p>
                                    
                                    <div class="mt-4">
                                        <div class="flex justify-between text-xs text-gray-500 dark:text-gray-400">
                                            <span>Progresso</span>
                                            <span>${progress}%</span>
                                        </div>
                                        <div class="w-full bg-gray-200 dark:bg-gray-700 rounded-full h-2 mt-1">
                                            <div class="bg-primary-500 h-2 rounded-full" style="width: ${progress}%"></div>
                                        </div>
                                    </div>
                                    
                                    <div class="grid grid-cols-2 gap-2 mt-4 text-sm">
                                        <div>
                                            <p class="text-xs text-gray-500 dark:text-gray-400">Início</p>
                                            <p>${startDate}</p>
                                        </div>
                                        <div>
                                            <p class="text-xs text-gray-500 dark:text-gray-400">Prazo</p>
                                            <p>${dueDate}</p>
                                        </div>
                                    </div>
                                    
                                    <div class="mt-4 flex items-center">
                                        <span class="text-xs ${priorityColor}" title="Prioridade">
                                            ${project.priority === 'high' ? '<i class="fas fa-angle-double-up"></i> Alta' : 
                                             project.priority === 'medium' ? '<i class="fas fa-equals"></i> Média' : 
                                             '<i class="fas fa-angle-double-down"></i> Baixa'}
                                        </span>
                                        <span class="text-xs text-gray-500 dark:text-gray-400 ml-4" title="Tarefas">
                                            <i class="fas fa-tasks"></i> ${totalTasks} tarefas (${completedTasks} concluídas)
                                        </span>
                                    </div>
                                </div>
                                
                                <div class="px-4 py-3 bg-gray-50 dark:bg-gray-700 flex justify-between">
                                    <button class="text-sm text-primary-500 hover:text-primary-700 project-view-btn" data-id="${project.id}">
                                        <i class="fas fa-eye"></i> Ver detalhes
                                    </button>
                                    <div>
                                        <button class="text-sm text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300 project-edit-btn ml-3" data-id="${project.id}">
                                            <i class="fas fa-edit"></i>
                                        </button>
                                        <button class="text-sm text-red-500 hover:text-red-700 dark:text-red-400 dark:hover:text-red-300 project-delete-btn ml-3" data-id="${project.id}">
                                            <i class="fas fa-trash-alt"></i>
                                        </button>
                                    </div>
                                </div>
                            </div>
                        `;
                    }).join('');
                    
                    // Adicionar eventos aos botões
                    document.querySelectorAll('.project-view-btn').forEach(btn => {
                        btn.addEventListener('click', function() {
                            const projectId = this.getAttribute('data-id');
                            openProjectDetail(projectId);
                        });
                    });
                    
                    document.querySelectorAll('.project-edit-btn').forEach(btn => {
                        btn.addEventListener('click', function() {
                            const projectId = this.getAttribute('data-id');
                            editProject(projectId);
                        });
                    });
                    
                    document.querySelectorAll('.project-delete-btn').forEach(btn => {
                        btn.addEventListener('click', function() {
                            const projectId = this.getAttribute('data-id');
                            confirmDeleteProject(projectId);
                        });
                    });
                } else {
                    projectsGrid.innerHTML = `
                        <div class="col-span-full text-center py-10">
                            <i class="fas fa-folder-open text-4xl text-gray-400 mb-3"></i>
                            <h3 class="text-lg font-medium mb-2">Nenhum projeto encontrado</h3>
                            <p class="text-gray-500 dark:text-gray-400 mb-4">Crie um novo projeto ou ajuste os filtros de busca.</p>
                            <button id="emptyStateProjectBtn" class="text-sm bg-primary-500 hover:bg-primary-600 text-white px-4 py-2 rounded-md shadow-sm">
                                <i class="fas fa-plus mr-1"></i> Novo Projeto
                            </button>
                        </div>
                    `;
                    
                    document.getElementById('emptyStateProjectBtn').addEventListener('click', function() {
                        openNewProjectModal();
                    });
                }
            }
        }

        // Renderizar tarefas
        function renderTasks() {
            const template = document.getElementById('tasks-template');
            document.getElementById('pageContent').innerHTML = template.innerHTML;
            
            const tasksTableBody = document.getElementById('tasksTableBody');
            const searchInput = document.getElementById('taskSearchInput');
            const projectFilter = document.getElementById('taskProjectFilter');
            const statusFilter = document.getElementById('taskStatusFilter');
            
            // Preencher o filtro de projetos
            projectFilter.innerHTML = '<option value="all">Todos os projetos</option>';
            store.projects.forEach(project => {
                projectFilter.innerHTML += `<option value="${project.id}">${project.name}</option>`;
            });
            
            // Adicionar eventos de filtro
            searchInput.addEventListener('input', filterTasks);
            projectFilter.addEventListener('change', filterTasks);
            statusFilter.addEventListener('change', filterTasks);
            
            // Adicionar evento ao botão nova tarefa
            document.getElementById('newTaskBtn').addEventListener('click', function() {
                openTaskModal();
            });
            
            // Carregar tarefas
            filterTasks();
            
            function filterTasks() {
                const searchTerm = searchInput.value.toLowerCase();
                const projectValue = projectFilter.value;
                const statusValue = statusFilter.value;
                
                let filteredTasks = [...store.tasks];
                
                // Aplicar filtro de busca
                if (searchTerm) {
                    filteredTasks = filteredTasks.filter(task => 
                        task.name.toLowerCase().includes(searchTerm) || 
                        (task.description && task.description.toLowerCase().includes(searchTerm))
                    );
                }
                
                // Aplicar filtro de projeto
                if (projectValue !== 'all') {
                    filteredTasks = filteredTasks.filter(task => task.projectId === projectValue);
                }
                
                // Aplicar filtro de status
                if (statusValue !== 'all') {
                    filteredTasks = filteredTasks.filter(task => task.status === statusValue);
                }
                
                // Ordenar por status e data de criação
                filteredTasks.sort((a, b) => {
                    // Priorizar tarefas pendentes
                    if (a.status !== b.status) {
                        if (a.status === 'todo') return -1;
                        if (b.status === 'todo') return 1;
                        if (a.status === 'in_progress') return -1;
                        if (b.status === 'in_progress') return 1;
                    }
                    // Em seguida, ordenar por data de criação (mais recentes primeiro)
                    return new Date(b.createdAt) - new Date(a.createdAt);
                });
                
                if (filteredTasks.length > 0) {
                    tasksTableBody.innerHTML = filteredTasks.map(task => {
                        // Obter o projeto
                        const project = store.projects.find(p => p.id === task.projectId);
                        
                        // Status da tarefa
                        let statusBadge, statusClass;
                        switch (task.status) {
                            case 'completed':
                                statusBadge = 'Concluída';
                                statusClass = 'bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-300';
                                break;
                            case 'in_progress':
                                statusBadge = 'Em Progresso';
                                statusClass = 'bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-300';
                                break;
                            default:
                                statusBadge = 'A Fazer';
                                statusClass = 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
                        }
                        
                        // Prioridade
                        let priorityBadge, priorityClass;
                        switch (task.priority) {
                            case 'high':
                                priorityBadge = 'Alta';
                                priorityClass = 'bg-red-100 text-red-800 dark:bg-red-900/30 dark:text-red-300';
                                break;
                            case 'medium':
                                priorityBadge = 'Média';
                                priorityClass = 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/30 dark:text-yellow-300';
                                break;
                            default:
                                priorityBadge = 'Baixa';
                                priorityClass = 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
                        }
                        
                        // Data formatada
                        const dueDate = task.dueDate ? formatDate(task.dueDate) : 'Não definido';
                        
                        return `
                            <tr class="hover:bg-gray-50 dark:hover:bg-gray-700/40">
                                <td class="px-6 py-4">
                                    <div class="flex items-center">
                                        <span class="font-medium">${task.name}</span>
                                    </div>
                                </td>
                                <td class="px-6 py-4">
                                    <span class="text-sm ${project ? '' : 'text-gray-400'}">
                                        ${project ? project.name : 'Projeto não encontrado'}
                                    </span>
                                </td>
                                <td class="px-6 py-4">
                                    <span class="px-2 py-1 rounded-full text-xs ${statusClass}">
                                        ${statusBadge}
                                    </span>
                                </td>
                                <td class="px-6 py-4">
                                    <span class="px-2 py-1 rounded-full text-xs ${priorityClass}">
                                        ${priorityBadge}
                                    </span>
                                </td>
                                <td class="px-6 py-4 text-sm">
                                    ${dueDate}
                                </td>
                                <td class="px-6 py-4">
                                    <div class="flex space-x-2">
                                        <button class="text-primary-500 hover:text-primary-700 dark:text-primary-400 dark:hover:text-primary-300 task-view-btn" data-id="${task.id}">
                                            <i class="fas fa-eye"></i>
                                        </button>
                                        <button class="text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300 task-edit-btn" data-id="${task.id}">
                                            <i class="fas fa-edit"></i>
                                        </button>
                                        <button class="text-red-500 hover:text-red-700 dark:text-red-400 dark:hover:text-red-300 task-delete-btn" data-id="${task.id}">
                                            <i class="fas fa-trash-alt"></i>
                                        </button>
                                    </div>
                                </td>
                            </tr>
                        `;
                    }).join('');
                    
                    // Adicionar eventos aos botões
                    document.querySelectorAll('.task-view-btn').forEach(btn => {
                        btn.addEventListener('click', function() {
                            const taskId = this.getAttribute('data-id');
                            viewTask(taskId);
                        });
                    });
                    
                    document.querySelectorAll('.task-edit-btn').forEach(btn => {
                        btn.addEventListener('click', function() {
                            const taskId = this.getAttribute('data-id');
                            editTask(taskId);
                        });
                    });
                    
                    document.querySelectorAll('.task-delete-btn').forEach(btn => {
                        btn.addEventListener('click', function() {
                            const taskId = this.getAttribute('data-id');
                            confirmDeleteTask(taskId);
                        });
                    });
                } else {
                    tasksTableBody.innerHTML = `
                        <tr>
                            <td colspan="6" class="px-6 py-10 text-center">
                                <i class="fas fa-tasks text-4xl text-gray-400 mb-3"></i>
                                <h3 class="text-lg font-medium mb-2">Nenhuma tarefa encontrada</h3>
                                <p class="text-gray-500 dark:text-gray-400 mb-4">Crie uma nova tarefa ou ajuste os filtros de busca.</p>
                                <button id="emptyStateTaskBtn" class="text-sm bg-primary-500 hover:bg-primary-600 text-white px-4 py-2 rounded-md shadow-sm">
                                    <i class="fas fa-plus mr-1"></i> Nova Tarefa
                                </button>
                            </td>
                        </tr>
                    `;
                    
                    document.getElementById('emptyStateTaskBtn').addEventListener('click', function() {
                        openTaskModal();
                    });
                }
            }
        }

        // Renderizar kanban
        function renderKanban() {
            const template = document.getElementById('kanban-template');
            document.getElementById('pageContent').innerHTML = template.innerHTML;
            
            const todoColumn = document.getElementById('todoColumn');
            const inProgressColumn = document.getElementById('inProgressColumn');
            const completedColumn = document.getElementById('completedColumn');
            const projectFilter = document.getElementById('kanbanProjectFilter');
            
            // Preencher o filtro de projetos
            projectFilter.innerHTML = '<option value="all">Todos os projetos</option>';
            store.projects.forEach(project => {
                projectFilter.innerHTML += `<option value="${project.id}">${project.name}</option>`;
            });
            
            // Adicionar evento ao filtro
            projectFilter.addEventListener('change', filterKanban);
            
            // Adicionar evento ao botão nova tarefa
            document.getElementById('newTaskBtnAlt').addEventListener('click', function() {
                openTaskModal();
            });
            
            // Habilitar drag and drop
            setupDragAndDrop();
            
            // Filtrar e exibir tarefas
            filterKanban();
            
            function filterKanban() {
                const projectValue = projectFilter.value;
                
                let filteredTasks = [...store.tasks];
                
                // Aplicar filtro de projeto
                if (projectValue !== 'all') {
                    filteredTasks = filteredTasks.filter(task => task.projectId === projectValue);
                }
                
                // Separar tarefas por status
                const todoTasks = filteredTasks.filter(task => task.status === 'todo');
                const inProgressTasks = filteredTasks.filter(task => task.status === 'in_progress');
                const completedTasks = filteredTasks.filter(task => task.status === 'completed');
                
                // Atualizar contadores
                document.getElementById('todoCount').textContent = todoTasks.length;
                document.getElementById('inProgressCount').textContent = inProgressTasks.length;
                document.getElementById('completedCount').textContent = completedTasks.length;
                
                // Ordenar por prioridade e data
                const sortTasks = tasks => tasks.sort((a, b) => {
                    // Primeiro por prioridade
                    const priorityOrder = { high: 0, medium: 1, low: 2 };
                    const priorityDiff = priorityOrder[a.priority] - priorityOrder[b.priority];
                    if (priorityDiff !== 0) return priorityDiff;
                    
                    // Depois por data de vencimento (mais próximas primeiro)
                    if (a.dueDate && b.dueDate) {
                        return new Date(a.dueDate) - new Date(b.dueDate);
                    }
                    
                    // Por fim, por data de criação (mais recentes primeiro)
                    return new Date(b.createdAt) - new Date(a.createdAt);
                });
                
                // Renderizar cada coluna
                renderColumn(todoColumn, sortTasks(todoTasks));
                renderColumn(inProgressColumn, sortTasks(inProgressTasks));
                renderColumn(completedColumn, sortTasks(completedTasks));
            }
            
            function renderColumn(column, tasks) {
                if (tasks.length === 0) {
                    column.innerHTML = `
                        <div class="flex flex-col items-center justify-center py-6 text-gray-400 dark:text-gray-500">
                            <i class="fas fa-inbox text-2xl mb-2"></i>
                            <p class="text-sm">Nenhuma tarefa</p>
                        </div>
                    `;
                    return;
                }
                
                column.innerHTML = tasks.map(task => {
                    // Obter o projeto
                    const project = store.projects.find(p => p.id === task.projectId);
                    
                    // Prioridade
                    let priorityClass;
                    switch (task.priority) {
                        case 'high':
                            priorityClass = 'border-l-4 border-red-500';
                            break;
                        case 'medium':
                            priorityClass = 'border-l-4 border-yellow-500';
                            break;
                        default:
                            priorityClass = 'border-l-4 border-gray-300 dark:border-gray-600';
                    }
                    
                    // Data formatada
                    const dueDate = task.dueDate ? formatDate(task.dueDate) : 'Não definido';
                    
                    // Status de data expirada
                    let dueDateClass = '';
                    if (task.dueDate) {
                        const today = new Date();
                        today.setHours(0, 0, 0, 0);
                        const dueDateTime = new Date(task.dueDate);
                        dueDateTime.setHours(0, 0, 0, 0);
                        
                        if (dueDateTime < today && task.status !== 'completed') {
                            dueDateClass = 'text-red-500 dark:text-red-400';
                        }
                    }
                    
                    return `
                        <div class="bg-white dark:bg-gray-800 rounded shadow-sm mb-2 ${priorityClass} drag-item" 
                             draggable="true" data-id="${task.id}" data-status="${task.status}">
                            <div class="p-3">
                                <div class="flex justify-between items-start">
                                    <h4 class="font-medium">${task.name}</h4>
                                </div>
                                <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">${project ? project.name : 'Projeto não encontrado'}</p>
                                <div class="flex justify-between items-center mt-3">
                                    <span class="text-xs ${dueDateClass}">
                                        <i class="far fa-calendar-alt mr-1"></i> ${dueDate}
                                    </span>
                                    <div>
                                        <button class="text-primary-500 hover:text-primary-700 dark:text-primary-400 dark:hover:text-primary-300 kanban-view-btn p-1" data-id="${task.id}">
                                            <i class="fas fa-eye"></i>
                                        </button>
                                        <button class="text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300 kanban-edit-btn p-1" data-id="${task.id}">
                                            <i class="fas fa-edit"></i>
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    `;
                }).join('');
                
                // Adicionar eventos aos botões
                column.querySelectorAll('.kanban-view-btn').forEach(btn => {
                    btn.addEventListener('click', function(e) {
                        e.stopPropagation(); // Evitar propagação para o item pai
                        const taskId = this.getAttribute('data-id');
                        viewTask(taskId);
                    });
                });
                
                column.querySelectorAll('.kanban-edit-btn').forEach(btn => {
                    btn.addEventListener('click', function(e) {
                        e.stopPropagation(); // Evitar propagação para o item pai
                        const taskId = this.getAttribute('data-id');
                        editTask(taskId);
                    });
                });
            }
            
            function setupDragAndDrop() {
                // Adicionar eventos às colunas de destino
                const dropZones = document.querySelectorAll('.drop-zone');
                dropZones.forEach(zone => {
                    zone.addEventListener('dragover', function(e) {
                        e.preventDefault();
                        this.classList.add('drag-over');
                    });
                    
                    zone.addEventListener('dragleave', function() {
                        this.classList.remove('drag-over');
                    });
                    
                    zone.addEventListener('drop', function(e) {
                        e.preventDefault();
                        this.classList.remove('drag-over');
                        
                        const taskId = e.dataTransfer.getData('text/plain');
                        const newStatus = this.getAttribute('data-status');
                        
                        // Atualizar status da tarefa
                        const task = store.tasks.find(t => t.id === taskId);
                        if (task && task.status !== newStatus) {
                            updateTask(taskId, { status: newStatus });
                            
                            // Atualizar Kanban
                            filterKanban();
                        }
                    });
                });
                
                // Adicionar eventos drag aos itens
                document.addEventListener('dragstart', function(e) {
                    if (e.target.classList.contains('drag-item')) {
                        e.dataTransfer.setData('text/plain', e.target.getAttribute('data-id'));
                        e.target.classList.add('opacity-50');
                    }
                });
                
                document.addEventListener('dragend', function(e) {
                    if (e.target.classList.contains('drag-item')) {
                        e.target.classList.remove('opacity-50');
                    }
                });
            }
        }

        // Renderizar timeline
        function renderTimeline() {
            const template = document.getElementById('timeline-template');
            document.getElementById('pageContent').innerHTML = template.innerHTML;
            
            const timelineContainer = document.getElementById('timelineContainer');
            const calendarContainer = document.getElementById('calendarContainer');
            const projectFilter = document.getElementById('timelineProjectFilter');
            const timelineViewBtn = document.getElementById('timelineView');
            const calendarViewBtn = document.getElementById('calendarView');
            
            // Preencher o filtro de projetos
            projectFilter.innerHTML = '<option value="all">Todos os projetos</option>';
            store.projects.forEach(project => {
                projectFilter.innerHTML += `<option value="${project.id}">${project.name}</option>`;
            });
            
            // Adicionar evento ao filtro
            projectFilter.addEventListener('change', function() {
                if (timelineContainer.classList.contains('hidden')) {
                    renderCalendar();
                } else {
                    renderTimelineContent();
                }
            });
            
            // Alternar entre visualizações
            timelineViewBtn.addEventListener('click', function() {
                if (timelineContainer.classList.contains('hidden')) {
                    timelineContainer.classList.remove('hidden');
                    calendarContainer.classList.add('hidden');
                    
                    // Atualizar estilos dos botões
                    timelineViewBtn.classList.add('bg-primary-500', 'text-white');
                    timelineViewBtn.classList.remove('bg-gray-200', 'dark:bg-gray-700', 'text-gray-700', 'dark:text-gray-300');
                    
                    calendarViewBtn.classList.remove('bg-primary-500', 'text-white');
                    calendarViewBtn.classList.add('bg-gray-200', 'dark:bg-gray-700', 'text-gray-700', 'dark:text-gray-300');
                    
                    renderTimelineContent();
                }
            });
            
            calendarViewBtn.addEventListener('click', function() {
                if (calendarContainer.classList.contains('hidden')) {
                    calendarContainer.classList.remove('hidden');
                    timelineContainer.classList.add('hidden');
                    
                    // Atualizar estilos dos botões
                    calendarViewBtn.classList.add('bg-primary-500', 'text-white');
                    calendarViewBtn.classList.remove('bg-gray-200', 'dark:bg-gray-700', 'text-gray-700', 'dark:text-gray-300');
                    
                    timelineViewBtn.classList.remove('bg-primary-500', 'text-white');
                    timelineViewBtn.classList.add('bg-gray-200', 'dark:bg-gray-700', 'text-gray-700', 'dark:text-gray-300');
                    
                    renderCalendar();
                }
            });
            
            // Configuração do calendário
            const calendarMonthYear = document.getElementById('calendarMonthYear');
            const prevMonth = document.getElementById('prevMonth');
            const nextMonth = document.getElementById('nextMonth');
            
            // Data atual para o calendário
            let currentCalendarDate = new Date();
            
            prevMonth.addEventListener('click', function() {
                currentCalendarDate.setMonth(currentCalendarDate.getMonth() - 1);
                renderCalendar();
            });
            
            nextMonth.addEventListener('click', function() {
                currentCalendarDate.setMonth(currentCalendarDate.getMonth() + 1);
                renderCalendar();
            });
            
            // Renderizar timeline inicial
            renderTimelineContent();
            
            function renderTimelineContent() {
                const projectId = projectFilter.value;
                
                // Itens do cronograma (projetos e tarefas)
                let timelineItems = [];
                
                // Projetos filtrados
                let filteredProjects = projectId === 'all' ? 
                    store.projects : 
                    store.projects.filter(p => p.id === projectId);
                
                // Adicionar projetos ao cronograma
                filteredProjects.forEach(project => {
                    if (project.startDate) {
                        timelineItems.push({
                            type: 'project_start',
                            id: project.id,
                            name: project.name,
                            date: new Date(project.startDate),
                            description: 'Início do projeto'
                        });
                    }
                    
                    if (project.dueDate) {
                        timelineItems.push({
                            type: 'project_due',
                            id: project.id,
                            name: project.name,
                            date: new Date(project.dueDate),
                            description: 'Prazo final do projeto'
                        });
                    }
                });
                
                // Tarefas filtradas
                let filteredTasks = projectId === 'all' ? 
                    store.tasks : 
                    store.tasks.filter(t => t.projectId === projectId);
                
                // Adicionar tarefas ao cronograma
                filteredTasks.forEach(task => {
                    if (task.dueDate) {
                        const project = store.projects.find(p => p.id === task.projectId);
                        
                        timelineItems.push({
                            type: 'task',
                            id: task.id,
                            name: task.name,
                            projectName: project ? project.name : 'Projeto desconhecido',
                            date: new Date(task.dueDate),
                            status: task.status,
                            priority: task.priority,
                            description: `Prazo da tarefa: ${task.name}`
                        });
                    }
                });
                
                // Ordenar por data
                timelineItems.sort((a, b) => a.date - b.date);
                
                if (timelineItems.length === 0) {
                    timelineContainer.innerHTML = `
                        <div class="text-center py-10">
                            <i class="fas fa-calendar-alt text-4xl text-gray-400 mb-3"></i>
                            <h3 class="text-lg font-medium mb-2">Nenhum evento no cronograma</h3>
                            <p class="text-gray-500 dark:text-gray-400 mb-4">Adicione projetos e tarefas com datas para visualizá-los aqui.</p>
                        </div>
                    `;
                    return;
                }
                
                // Calcular intervalo de datas
                const earliestDate = new Date(Math.min(...timelineItems.map(item => item.date)));
                const latestDate = new Date(Math.max(...timelineItems.map(item => item.date)));
                
                // Adicionar margem de 7 dias antes e depois
                earliestDate.setDate(earliestDate.getDate() - 7);
                latestDate.setDate(latestDate.getDate() + 7);
                
                // Calcular número total de dias
                const totalDays = Math.ceil((latestDate - earliestDate) / (1000 * 60 * 60 * 24));
                
                // Gerar eixo temporal
                let timelineHTML = `
                    <div class="relative pb-10">
                        <!-- Eixo do tempo -->
                        <div class="absolute left-0 right-0 h-1 bg-gray-200 dark:bg-gray-700 top-10"></div>
                        
                        <!-- Marcador de hoje -->
                        <div class="absolute top-6 bottom-0 border-l-2 border-red-500 dark:border-red-400 z-10" 
                             style="left: ${calculatePositionInTimeline(new Date(), earliestDate, totalDays)}%;">
                            <div class="bg-red-500 text-white text-xs px-1.5 py-0.5 rounded absolute -left-9 -top-6">
                                Hoje
                            </div>
                        </div>
                        
                        <!-- Marcadores de mês -->
                        ${generateMonthMarkers(earliestDate, latestDate, totalDays)}
                        
                        <!-- Itens do cronograma -->
                        <div class="relative pt-20">
                `;
                
                // Agrupar itens por projeto para melhor visualização
                const projectGroups = {};
                timelineItems.forEach(item => {
                    const projectId = item.type === 'task' ? 
                        store.tasks.find(t => t.id === item.id)?.projectId : 
                        item.id;
                    
                    if (!projectGroups[projectId]) {
                        projectGroups[projectId] = [];
                    }
                    
                    projectGroups[projectId].push(item);
                });
                
                // Criar uma linha para cada projeto
                Object.entries(projectGroups).forEach(([projectId, items], index) => {
                    const project = store.projects.find(p => p.id === projectId);
                    const projectName = project ? project.name : 'Projeto não encontrado';
                    
                    timelineHTML += `
                        <div class="mb-8">
                            <div class="text-sm font-medium mb-2">${projectName}</div>
                            <div class="relative h-16 border-b border-gray-100 dark:border-gray-800">
                    `;
                    
                    // Posicionar itens na timeline
                    items.forEach(item => {
                        const position = calculatePositionInTimeline(item.date, earliestDate, totalDays);
                        
                        // Definir ícone e cores baseado no tipo
                        let icon, bgColor, textColor;
                        
                        if (item.type === 'project_start') {
                            icon = 'play-circle';
                            bgColor = 'bg-green-100 dark:bg-green-900/30';
                            textColor = 'text-green-800 dark:text-green-200';
                        } else if (item.type === 'project_due') {
                            icon = 'flag-checkered';
                            bgColor = 'bg-blue-100 dark:bg-blue-900/30';
                            textColor = 'text-blue-800 dark:text-blue-200';
                        } else if (item.type === 'task') {
                            icon = 'tasks';
                            
                            if (item.status === 'completed') {
                                bgColor = 'bg-green-100 dark:bg-green-900/30';
                                textColor = 'text-green-800 dark:text-green-200';
                            } else if (item.priority === 'high') {
                                bgColor = 'bg-red-100 dark:bg-red-900/30';
                                textColor = 'text-red-800 dark:text-red-200';
                            } else if (item.priority === 'medium') {
                                bgColor = 'bg-yellow-100 dark:bg-yellow-900/30';
                                textColor = 'text-yellow-800 dark:text-yellow-200';
                            } else {
                                bgColor = 'bg-gray-100 dark:bg-gray-700';
                                textColor = 'text-gray-800 dark:text-gray-200';
                            }
                        }
                        
                        timelineHTML += `
                            <div class="absolute transform -translate-x-1/2" style="left: ${position}%; top: 0;">
                                <!-- Linha conectora -->
                                <div class="absolute left-1/2 -ml-px w-0.5 bg-gray-200 dark:bg-gray-700 h-6 -top-6"></div>
                                
                                <!-- Marcador -->
                                <div class="absolute -top-8 left-1/2 transform -translate-x-1/2 w-4 h-4 rounded-full ${bgColor}"></div>
                                
                                <!-- Card do evento -->
                                <div class="absolute top-0 -translate-x-1/2 w-48 cursor-pointer timeline-item"
                                     data-id="${item.id}" data-type="${item.type}">
                                    <div class="${bgColor} ${textColor} rounded-lg shadow-sm border border-gray-200 dark:border-gray-700 p-2 text-xs">
                                        <div class="flex items-center">
                                            <i class="fas fa-${icon} mr-1"></i>
                                            <span class="font-medium truncate">${item.name}</span>
                                        </div>
                                        <p class="text-xs mt-1">${formatDate(item.date)}</p>
                                        <p class="text-xs mt-1 truncate">${item.description}</p>
                                    </div>
                                </div>
                            </div>
                        `;
                    });
                    
                    timelineHTML += `
                            </div>
                        </div>
                    `;
                });
                
                timelineHTML += `
                        </div>
                    </div>
                `;
                
                timelineContainer.innerHTML = timelineHTML;
                
                // Adicionar eventos aos itens da timeline
                document.querySelectorAll('.timeline-item').forEach(item => {
                    item.addEventListener('click', function() {
                        const itemType = this.getAttribute('data-type');
                        const itemId = this.getAttribute('data-id');
                        
                        if (itemType === 'task') {
                            viewTask(itemId);
                        } else {
                            // Projeto
                            openProjectDetail(itemId);
                        }
                    });
                });
            }
            
            function renderCalendar() {
                const projectId = projectFilter.value;
                const month = currentCalendarDate.getMonth();
                const year = currentCalendarDate.getFullYear();
                
                // Atualizar título do mês
                const monthNames = ['Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho', 
                                    'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'];
                calendarMonthYear.textContent = `${monthNames[month]} ${year}`;
                
                // Obter primeiro dia do mês
                const firstDay = new Date(year, month, 1);
                
                // Obter último dia do mês
                const lastDay = new Date(year, month + 1, 0);
                
                // Dia da semana do primeiro dia (0-6, 0 = Domingo)
                const firstDayOfWeek = firstDay.getDay();
                
                // Número de dias no mês
                const daysInMonth = lastDay.getDate();
                
                // Grid do calendário
                const calendarGrid = document.getElementById('calendarGrid');
                
                // Limpar dias anteriores (mas manter o cabeçalho dos dias da semana)
                const daysElements = calendarGrid.querySelectorAll('.calendar-day');
                daysElements.forEach(day => day.remove());
                
                // Obter eventos do mês - projetos
                let calendarEvents = [];
                
                // Filtrar projetos
                let filteredProjects = projectId === 'all' ? 
                    store.projects : 
                    store.projects.filter(p => p.id === projectId);
                
                // Adicionar eventos de início e fim de projeto
                filteredProjects.forEach(project => {
                    if (project.startDate) {
                        const startDate = new Date(project.startDate);
                        calendarEvents.push({
                            date: startDate,
                            type: 'project_start',
                            id: project.id,
                            title: `Início: ${project.name}`,
                            color: 'green'
                        });
                    }
                    
                    if (project.dueDate) {
                        const dueDate = new Date(project.dueDate);
                        calendarEvents.push({
                            date: dueDate,
                            type: 'project_due',
                            id: project.id,
                            title: `Prazo: ${project.name}`,
                            color: 'blue'
                        });
                    }
                });
                
                // Filtrar tarefas
                let filteredTasks = projectId === 'all' ? 
                    store.tasks : 
                    store.tasks.filter(t => t.projectId === projectId);
                
                // Adicionar datas de tarefas
                filteredTasks.forEach(task => {
                    if (task.dueDate) {
                        const dueDate = new Date(task.dueDate);
                        
                        let color;
                        if (task.status === 'completed') {
                            color = 'green';
                        } else if (task.priority === 'high') {
                            color = 'red';
                        } else if (task.priority === 'medium') {
                            color = 'yellow';
                        } else {
                            color = 'gray';
                        }
                        
                        calendarEvents.push({
                            date: dueDate,
                            type: 'task',
                            id: task.id,
                            title: task.name,
                            status: task.status,
                            color: color
                        });
                    }
                });
                
                // Gerar dias do mês anterior para preencher a primeira semana
                for (let i = 0; i < firstDayOfWeek; i++) {
                    const day = new Date(year, month, -firstDayOfWeek + i + 1);
                    const dayElement = createDayElement(day, true);
                    calendarGrid.appendChild(dayElement);
                }
                
                // Gerar dias do mês atual
                for (let i = 1; i <= daysInMonth; i++) {
                    const day = new Date(year, month, i);
                    const dayEvents = calendarEvents.filter(event => 
                        event.date.getDate() === day.getDate() && 
                        event.date.getMonth() === day.getMonth() && 
                        event.date.getFullYear() === day.getFullYear()
                    );
                    
                    const dayElement = createDayElement(day, false, dayEvents);
                    calendarGrid.appendChild(dayElement);
                }
                
                // Calcular quantos dias do próximo mês precisamos para completar o grid
                const lastDayOfWeek = lastDay.getDay();
                const remainingDays = 6 - lastDayOfWeek;
                
                // Gerar dias do próximo mês
                for (let i = 1; i <= remainingDays; i++) {
                    const day = new Date(year, month + 1, i);
                    const dayElement = createDayElement(day, true);
                    calendarGrid.appendChild(dayElement);
                }
                
                // Esconder o container de eventos do dia
                document.getElementById('dayEventsContainer').classList.add('hidden');
            }
            
            function createDayElement(date, isOtherMonth, events = []) {
                const dayElement = document.createElement('div');
                dayElement.className = 'calendar-day border border-gray-200 dark:border-gray-700 p-1';
                dayElement.setAttribute('data-date', date.toISOString().split('T')[0]);
                
                if (isOtherMonth) {
                    dayElement.classList.add('other-month');
                }
                
                // Verificar se é hoje
                const today = new Date();
                if (date.getDate() === today.getDate() && 
                    date.getMonth() === today.getMonth() && 
                    date.getFullYear() === today.getFullYear()) {
                    dayElement.classList.add('today');
                    dayElement.classList.add('border-primary-500', 'dark:border-primary-400');
                }
                
                // Estrutura básica do dia
                dayElement.innerHTML = `
                    <div class="flex justify-between items-center mb-1">
                        <span class="text-sm font-medium ${isOtherMonth ? 'text-gray-400 dark:text-gray-600' : ''}">${date.getDate()}</span>
                    </div>
                    <div class="day-events overflow-hidden flex-1">
                        <!-- Events will be dynamically added here -->
                    </div>
                `;
                
                const dayEventsContainer = dayElement.querySelector('.day-events');
                
                // Se há eventos neste dia
                if (events.length > 0) {
                    dayElement.classList.add('has-events');
                    
                    // Adicionar no máximo 2 eventos visíveis
                    const visibleEvents = events.slice(0, 2);
                    visibleEvents.forEach(event => {
                        const eventElement = document.createElement('div');
                        eventElement.className = `event-badge bg-${event.color}-100 dark:bg-${event.color}-900/30 text-${event.color}-800 dark:text-${event.color}-300`;
                        eventElement.textContent = event.title;
                        dayEventsContainer.appendChild(eventElement);
                    });
                    
                    // Se houver mais eventos, mostrar contador
                    if (events.length > 2) {
                        const moreElement = document.createElement('div');
                        moreElement.className = 'event-badge bg-gray-100 dark:bg-gray-700 text-gray-800 dark:text-gray-300 text-center';
                        moreElement.textContent = `+${events.length - 2} mais`;
                        dayEventsContainer.appendChild(moreElement);
                    }
                    
                    // Adicionar indicadores na parte inferior
                    const indicatorElement = document.createElement('div');
                    indicatorElement.className = 'event-indicator';
                    
                    events.slice(0, 3).forEach(event => {
                        const dotElement = document.createElement('div');
                        dotElement.className = `dot bg-${event.color}-500 dark:bg-${event.color}-400`;
                        indicatorElement.appendChild(dotElement);
                    });
                    
                    dayElement.appendChild(indicatorElement);
                    
                    // Criar preview dos eventos (mostrado no hover)
                    const previewElement = document.createElement('div');
                    previewElement.className = 'event-preview';
                    previewElement.innerHTML = `
                        <p class="text-xs font-medium mb-1">${date.getDate()} de ${monthNames[date.getMonth()]}</p>
                        <div class="space-y-1">
                            ${events.map(event => `
                                <div class="text-xs bg-${event.color}-100 dark:bg-${event.color}-900/30 text-${event.color}-800 dark:text-${event.color}-300 p-1 rounded">
                                    ${event.title}
                                </div>
                            `).join('')}
                        </div>
                    `;
                    dayElement.appendChild(previewElement);
                }
                
                // Adicionar evento de clique para mostrar eventos do dia
                dayElement.addEventListener('click', function() {
                    const dateStr = this.getAttribute('data-date');
                    showDayEvents(dateStr, events);
                });
                
                return dayElement;
            }
            
            function showDayEvents(dateStr, events) {
                // Limpar seleção anterior
                document.querySelectorAll('.calendar-day.selected').forEach(day => {
                    day.classList.remove('selected');
                });
                
                // Selecionar o dia clicado
                const clickedDay = document.querySelector(`.calendar-day[data-date="${dateStr}"]`);
                if (clickedDay) {
                    clickedDay.classList.add('selected');
                }
                
                const date = new Date(dateStr);
                const dateDisplay = date.toLocaleDateString('pt-BR', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
                document.getElementById('dayEventsTitle').textContent = `Eventos de ${dateDisplay}`;
                
                // Obter todos os eventos para este dia
                const dayEvents = [];
                const projectId = projectFilter.value;
                
                // Filtrar projetos
                let filteredProjects = projectId === 'all' ? 
                    store.projects : 
                    store.projects.filter(p => p.id === projectId);
                
                // Adicionar eventos de projeto
                filteredProjects.forEach(project => {
                    if (project.startDate) {
                        const startDate = new Date(project.startDate);
                        startDate.setHours(0, 0, 0, 0);
                        
                        const eventDate = new Date(dateStr);
                        eventDate.setHours(0, 0, 0, 0);
                        
                        if (startDate.getTime() === eventDate.getTime()) {
                            dayEvents.push({
                                type: 'project_start',
                                id: project.id,
                                name: project.name,
                                description: 'Início do projeto',
                                color: 'green',
                                project: project
                            });
                        }
                    }
                    
                    if (project.dueDate) {
                        const dueDate = new Date(project.dueDate);
                        dueDate.setHours(0, 0, 0, 0);
                        
                        const eventDate = new Date(dateStr);
                        eventDate.setHours(0, 0, 0, 0);
                        
                        if (dueDate.getTime() === eventDate.getTime()) {
                            dayEvents.push({
                                type: 'project_due',
                                id: project.id,
                                name: project.name,
                                description: 'Prazo final do projeto',
                                color: 'blue',
                                project: project
                            });
                        }
                    }
                });
                
                // Filtrar tarefas
                let filteredTasks = projectId === 'all' ? 
                    store.tasks : 
                    store.tasks.filter(t => t.projectId === projectId);
                
                // Adicionar eventos de tarefa
                filteredTasks.forEach(task => {
                    if (task.dueDate) {
                        const dueDate = new Date(task.dueDate);
                        dueDate.setHours(0, 0, 0, 0);
                        
                        const eventDate = new Date(dateStr);
                        eventDate.setHours(0, 0, 0, 0);
                        
                        if (dueDate.getTime() === eventDate.getTime()) {
                            const project = store.projects.find(p => p.id === task.projectId);
                            
                            let color;
                            if (task.status === 'completed') {
                                color = 'green';
                            } else if (task.priority === 'high') {
                                color = 'red';
                            } else if (task.priority === 'medium') {
                                color = 'yellow';
                            } else {
                                color = 'gray';
                            }
                            
                            dayEvents.push({
                                type: 'task',
                                id: task.id,
                                name: task.name,
                                projectName: project ? project.name : 'Projeto desconhecido',
                                description: 'Prazo da tarefa',
                                status: task.status,
                                color: color,
                                task: task,
                                project: project
                            });
                        }
                    }
                });
                
                // Exibir eventos
                const dayEventsList = document.getElementById('dayEventsList');
                
                if (dayEvents.length === 0) {
                    dayEventsList.innerHTML = `<p class="text-gray-500 dark:text-gray-400">Nenhum evento neste dia.</p>`;
                } else {
                    dayEventsList.innerHTML = dayEvents.map(event => {
                        // Ícone com base no tipo
                        let icon;
                        if (event.type === 'project_start') {
                            icon = 'play-circle';
                        } else if (event.type === 'project_due') {
                            icon = 'flag-checkered';
                        } else {
                            icon = 'tasks';
                        }
                        
                        // Para tarefas, mostrar status
                        let statusBadge = '';
                        if (event.type === 'task') {
                            let statusClass, statusText;
                            switch (event.status) {
                                case 'todo':
                                    statusClass = 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
                                    statusText = 'A Fazer';
                                    break;
                                case 'in_progress':
                                    statusClass = 'bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-300';
                                    statusText = 'Em Progresso';
                                    break;
                                case 'completed':
                                    statusClass = 'bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-300';
                                    statusText = 'Concluída';
                                    break;
                            }
                            statusBadge = `<span class="ml-2 px-1.5 py-0.5 text-xs rounded-full ${statusClass}">${statusText}</span>`;
                        }
                        
                        // Link do projeto para tarefas
                        let projectLink = '';
                        if (event.type === 'task' && event.project) {
                            projectLink = `<div class="text-xs text-gray-500 dark:text-gray-400">Projeto: ${event.projectName}</div>`;
                        }
                        
                        return `
                            <div class="bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-md p-3 calendar-event-item"
                                 data-type="${event.type}" data-id="${event.id}">
                                <div class="flex items-start">
                                    <div class="flex-shrink-0 w-8 h-8 bg-${event.color}-100 dark:bg-${event.color}-900/30 rounded-full flex items-center justify-center">
                                        <i class="fas fa-${icon} text-${event.color}-500 dark:text-${event.color}-400"></i>
                                    </div>
                                    <div class="ml-3 flex-1">
                                        <div class="flex items-center">
                                            <h4 class="font-medium">${event.name}</h4>
                                            ${statusBadge}
                                        </div>
                                        <p class="text-sm text-gray-600 dark:text-gray-400">${event.description}</p>
                                        ${projectLink}
                                    </div>
                                </div>
                            </div>
                        `;
                    }).join('');
                    
                    // Adicionar eventos aos itens do calendário
                    document.querySelectorAll('.calendar-event-item').forEach(item => {
                        item.addEventListener('click', function() {
                            const itemType = this.getAttribute('data-type');
                            const itemId = this.getAttribute('data-id');
                            
                            if (itemType === 'task') {
                                viewTask(itemId);
                            } else {
                                openProjectDetail(itemId);
                            }
                        });
                    });
                }
                
                // Mostrar container de eventos
                document.getElementById('dayEventsContainer').classList.remove('hidden');
            }
            
            function calculatePositionInTimeline(date, startDate, totalDays) {
                const diff = date - startDate;
                const days = diff / (1000 * 60 * 60 * 24);
                return (days / totalDays) * 100;
            }
            
            function generateMonthMarkers(startDate, endDate, totalDays) {
                let html = '';
                const monthNames = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];
                
                // Clonar para não modificar as datas originais
                let currentDate = new Date(startDate);
                
                // Ir para o primeiro dia do mês
                currentDate.setDate(1);
                
                // Se o mês já começou, avançar para o próximo
                if (currentDate.getTime() < startDate.getTime()) {
                    currentDate.setMonth(currentDate.getMonth() + 1);
                }
                
                while (currentDate <= endDate) {
                    const position = calculatePositionInTimeline(currentDate, startDate, totalDays);
                    const monthName = monthNames[currentDate.getMonth()];
                    const year = currentDate.getFullYear();
                    
                    html += `
                        <div class="absolute top-0 transform -translate-x-1/2" style="left: ${position}%">
                            <div class="border-l border-gray-300 dark:border-gray-600 h-4"></div>
                            <div class="text-xs text-gray-500 dark:text-gray-400 mt-1">${monthName} ${year}</div>
                        </div>
                    `;
                    
                    // Avançar para o próximo mês
                    currentDate.setMonth(currentDate.getMonth() + 1);
                }
                
                return html;
            }
        }

        // Renderizar logs
        function renderLogs() {
            const template = document.getElementById('logs-template');
            document.getElementById('pageContent').innerHTML = template.innerHTML;
            
            const logsTimeline = document.getElementById('logsTimeline');
            const logTypeFilter = document.getElementById('logTypeFilter');
            
            // Adicionar evento ao filtro
            logTypeFilter.addEventListener('change', filterLogs);
            
            // Carregar logs
            filterLogs();
            
            function filterLogs() {
                const typeValue = logTypeFilter.value;
                
                let filteredLogs = [...store.logs];
                
                // Aplicar filtro de tipo
                if (typeValue !== 'all') {
                    filteredLogs = filteredLogs.filter(log => log.type === typeValue);
                }
                
                // Ordenar por data (mais recentes primeiro)
                filteredLogs.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
                
                if (filteredLogs.length > 0) {
                    logsTimeline.innerHTML = filteredLogs.map(log => {
                        // Formatar data
                        const date = new Date(log.createdAt);
                        const formattedDate = date.toLocaleDateString('pt-BR', { 
                            day: '2-digit', 
                            month: '2-digit', 
                            year: 'numeric'
                        });
                        const formattedTime = date.toLocaleTimeString('pt-BR', {
                            hour: '2-digit',
                            minute: '2-digit'
                        });
                        
                        // Determinar ícone e cores com base no tipo
                        let icon, bgColor, textColor;
                        
                        switch (log.type) {
                            case 'project_created':
                                icon = 'folder-plus';
                                bgColor = 'bg-blue-100 dark:bg-blue-900/30';
                                textColor = 'text-blue-800 dark:text-blue-300';
                                break;
                            case 'project_updated':
                                icon = 'edit';
                                bgColor = 'bg-blue-100 dark:bg-blue-900/30';
                                textColor = 'text-blue-800 dark:text-blue-300';
                                break;
                            case 'project_deleted':
                                icon = 'folder-minus';
                                bgColor = 'bg-red-100 dark:bg-red-900/30';
                                textColor = 'text-red-800 dark:text-red-300';
                                break;
                            case 'task_created':
                                icon = 'tasks';
                                bgColor = 'bg-purple-100 dark:bg-purple-900/30';
                                textColor = 'text-purple-800 dark:text-purple-300';
                                break;
                            case 'task_updated':
                                icon = 'edit';
                                bgColor = 'bg-purple-100 dark:bg-purple-900/30';
                                textColor = 'text-purple-800 dark:text-purple-300';
                                break;
                            case 'task_deleted':
                                icon = 'trash-alt';
                                bgColor = 'bg-red-100 dark:bg-red-900/30';
                                textColor = 'text-red-800 dark:text-red-300';
                                break;
                            case 'task_completed':
                                icon = 'check-circle';
                                bgColor = 'bg-green-100 dark:bg-green-900/30';
                                textColor = 'text-green-800 dark:text-green-300';
                                break;
                            case 'idea_created':
                                icon = 'lightbulb';
                                bgColor = 'bg-yellow-100 dark:bg-yellow-900/30';
                                textColor = 'text-yellow-800 dark:text-yellow-300';
                                break;
                            case 'idea_updated':
                                icon = 'edit';
                                bgColor = 'bg-yellow-100 dark:bg-yellow-900/30';
                                textColor = 'text-yellow-800 dark:text-yellow-300';
                                break;
                            case 'idea_deleted':
                                icon = 'trash-alt';
                                bgColor = 'bg-red-100 dark:bg-red-900/30';
                                textColor = 'text-red-800 dark:text-red-300';
                                break;
                            default:
                                icon = 'history';
                                bgColor = 'bg-gray-100 dark:bg-gray-700';
                                textColor = 'text-gray-800 dark:text-gray-300';
                        }
                        
                        // Adicionar link para o projeto ou tarefa se existir
                        let linkHtml = '';
                        if (log.projectId) {
                            const project = store.projects.find(p => p.id === log.projectId);
                            if (project) {
                                linkHtml = `
                                    <a href="#" class="text-primary-500 hover:text-primary-700 dark:text-primary-400 dark:hover:text-primary-300 log-project-link mt-1 inline-block" data-id="${log.projectId}">
                                        Ver projeto
                                    </a>
                                `;
                            }
                        } else if (log.taskId) {
                            const task = store.tasks.find(t => t.id === log.taskId);
                            if (task) {
                                linkHtml = `
                                    <a href="#" class="text-primary-500 hover:text-primary-700 dark:text-primary-400 dark:hover:text-primary-300 log-task-link mt-1 inline-block" data-id="${log.taskId}">
                                        Ver tarefa
                                    </a>
                                `;
                            }
                        }
                        
                        return `
                            <div class="mb-8 relative">
                                <!-- Marcador na timeline -->
                                <div class="absolute -left-14 mt-1.5">
                                    <div class="h-8 w-8 rounded-full ${bgColor} flex items-center justify-center">
                                        <i class="fas fa-${icon} ${textColor}"></i>
                                    </div>
                                </div>
                                
                                <!-- Conteúdo do log -->
                                <div class="bg-white dark:bg-gray-800 shadow-sm rounded-lg p-4 border border-gray-200 dark:border-gray-700">
                                    <div class="flex justify-between items-start">
                                        <div>
                                            <p class="font-medium">${log.message}</p>
                                            ${linkHtml}
                                        </div>
                                        <span class="text-xs text-gray-500 dark:text-gray-400 whitespace-nowrap ml-4">
                                            ${formattedDate} - ${formattedTime}
                                        </span>
                                    </div>
                                </div>
                            </div>
                        `;
                    }).join('');
                    
                    // Adicionar eventos aos links
                    document.querySelectorAll('.log-project-link').forEach(link => {
                        link.addEventListener('click', function(e) {
                            e.preventDefault();
                            const projectId = this.getAttribute('data-id');
                            openProjectDetail(projectId);
                        });
                    });
                    
                    document.querySelectorAll('.log-task-link').forEach(link => {
                        link.addEventListener('click', function(e) {
                            e.preventDefault();
                            const taskId = this.getAttribute('data-id');
                            viewTask(taskId);
                        });
                    });
                } else {
                    logsTimeline.innerHTML = `
                        <div class="text-center py-10 -ml-6">
                            <i class="fas fa-history text-4xl text-gray-400 mb-3"></i>
                            <h3 class="text-lg font-medium mb-2">Nenhum registro encontrado</h3>
                            <p class="text-gray-500 dark:text-gray-400">Use o sistema para gerar registros de atividades.</p>
                        </div>
                    `;
                }
            }
        }

        // Renderizar ideias
        function renderIdeas() {
            const template = document.getElementById('ideas-template');
            document.getElementById('pageContent').innerHTML = template.innerHTML;
            
            const ideasGrid = document.getElementById('ideasGrid');
            const searchInput = document.getElementById('ideaSearchInput');
            const categoryFilter = document.getElementById('ideaCategoryFilter');
            
            // Adicionar eventos de filtro
            searchInput.addEventListener('input', filterIdeas);
            categoryFilter.addEventListener('change', filterIdeas);
            
            // Adicionar evento ao botão nova ideia
            document.getElementById('newIdeaBtn').addEventListener('click', function() {
                openIdeaModal();
            });
            
            // Carregar ideias
            filterIdeas();
            
            function filterIdeas() {
                const searchTerm = searchInput.value.toLowerCase();
                const categoryValue = categoryFilter.value;
                
                let filteredIdeas = [...store.ideas];
                
                // Aplicar filtro de busca
                if (searchTerm) {
                    filteredIdeas = filteredIdeas.filter(idea => 
                        idea.title.toLowerCase().includes(searchTerm) || 
                        (idea.description && idea.description.toLowerCase().includes(searchTerm)) ||
                        (idea.tags && idea.tags.some(tag => tag.toLowerCase().includes(searchTerm)))
                    );
                }
                
                // Aplicar filtro de categoria
                if (categoryValue !== 'all') {
                    filteredIdeas = filteredIdeas.filter(idea => idea.category === categoryValue);
                }
                
                // Ordenar por data de criação (mais recentes primeiro)
                filteredIdeas.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
                
                if (filteredIdeas.length > 0) {
                    ideasGrid.innerHTML = filteredIdeas.map(idea => {
                        // Formatar data
                        const createdDate = formatDate(idea.createdAt);
                        
                        // Definir cor com base na categoria
                        let categoryIcon, categoryColor;
                        switch (idea.category) {
                            case 'feature':
                                categoryIcon = 'puzzle-piece';
                                categoryColor = 'text-blue-500 dark:text-blue-400';
                                break;
                            case 'improvement':
                                categoryIcon = 'chart-line';
                                categoryColor = 'text-green-500 dark:text-green-400';
                                break;
                            case 'project':
                                categoryIcon = 'project-diagram';
                                categoryColor = 'text-purple-500 dark:text-purple-400';
                                break;
                            case 'process':
                                categoryIcon = 'cogs';
                                categoryColor = 'text-orange-500 dark:text-orange-400';
                                break;
                            default:
                                categoryIcon = 'tag';
                                categoryColor = 'text-gray-500 dark:text-gray-400';
                        }
                        
                        // Renderizar tags
                        const tagsHtml = idea.tags ? 
                            `<div class="flex flex-wrap gap-1 mt-3">
                                ${idea.tags.map(tag => `
                                    <span class="text-xs bg-gray-100 dark:bg-gray-700 text-gray-600 dark:text-gray-300 px-2 py-0.5 rounded-full">
                                        ${tag}
                                    </span>
                                `).join('')}
                            </div>` : '';
                        
                        return `
                            <div class="bg-white dark:bg-gray-800 rounded-lg shadow overflow-hidden border border-gray-200 dark:border-gray-700 hover:shadow-md transition-shadow">
                                <div class="p-4">
                                    <div class="flex justify-between items-start">
                                        <h3 class="text-lg font-semibold flex-1">${idea.title}</h3>
                                        <span class="${categoryColor}">
                                            <i class="fas fa-${categoryIcon}"></i>
                                        </span>
                                    </div>
                                    <p class="text-sm text-gray-600 dark:text-gray-400 mt-2 line-clamp-3">${idea.description}</p>
                                    ${tagsHtml}
                                </div>
                                
                                <div class="px-4 py-3 bg-gray-50 dark:bg-gray-700 flex justify-between items-center">
                                    <span class="text-xs text-gray-500 dark:text-gray-400">
                                        <i class="far fa-calendar mr-1"></i> ${createdDate}
                                    </span>
                                    <div>
                                        <button class="text-sm text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300 idea-edit-btn ml-3" data-id="${idea.id}">
                                            <i class="fas fa-edit"></i>
                                        </button>
                                        <button class="text-sm text-red-500 hover:text-red-700 dark:text-red-400 dark:hover:text-red-300 idea-delete-btn ml-3" data-id="${idea.id}">
                                            <i class="fas fa-trash-alt"></i>
                                        </button>
                                    </div>
                                </div>
                            </div>
                        `;
                    }).join('');
                    
                    // Adicionar eventos aos botões
                    document.querySelectorAll('.idea-edit-btn').forEach(btn => {
                        btn.addEventListener('click', function() {
                            const ideaId = this.getAttribute('data-id');
                            editIdea(ideaId);
                        });
                    });
                    
                    document.querySelectorAll('.idea-delete-btn').forEach(btn => {
                        btn.addEventListener('click', function() {
                            const ideaId = this.getAttribute('data-id');
                            confirmDeleteIdea(ideaId);
                        });
                    });
                } else {
                    ideasGrid.innerHTML = `
                        <div class="col-span-full text-center py-10">
                            <i class="fas fa-lightbulb text-4xl text-gray-400 mb-3"></i>
                            <h3 class="text-lg font-medium mb-2">Nenhuma ideia encontrada</h3>
                            <p class="text-gray-500 dark:text-gray-400 mb-4">Registre uma nova ideia ou ajuste os filtros de busca.</p>
                            <button id="emptyStateIdeaBtn" class="text-sm bg-primary-500 hover:bg-primary-600 text-white px-4 py-2 rounded-md shadow-sm">
                                <i class="fas fa-plus mr-1"></i> Nova Ideia
                            </button>
                        </div>
                    `;
                    
                    document.getElementById('emptyStateIdeaBtn').addEventListener('click', function() {
                        openIdeaModal();
                    });
                }
            }
        }

        // MODAIS E AÇÕES

        // Abrir modal de novo projeto
        function openNewProjectModal(projectData = null) {
            const modal = document.getElementById('newProjectModal');
            const form = document.getElementById('newProjectForm');
            const title = modal.querySelector('h3');
            const submitBtn = form.querySelector('button[type="submit"]');
            
            // Limpar dados anteriores
            form.reset();
            
            // Configurar para novo projeto ou edição
            if (projectData) {
                title.textContent = 'Editar Projeto';
                submitBtn.textContent = 'Atualizar Projeto';
                
                // Preencher formulário com dados existentes
                document.getElementById('projectName').value = projectData.name || '';
                document.getElementById('projectDescription').value = projectData.description || '';
                document.getElementById('projectWhy').value = projectData.why || '';
                document.getElementById('projectWhere').value = projectData.where || '';
                document.getElementById('projectWho').value = projectData.who || '';
                document.getElementById('projectHow').value = projectData.how || '';
                document.getElementById('projectHowMuch').value = projectData.howMuch || '';
                document.getElementById('projectStartDate').value = projectData.startDate || '';
                document.getElementById('projectDueDate').value = projectData.dueDate || '';
                document.getElementById('projectPriority').value = projectData.priority || 'medium';
                
                // Adicionar ID ao formulário para identificar na submissão
                form.setAttribute('data-project-id', projectData.id);
            } else {
                title.textContent = 'Novo Projeto';
                submitBtn.textContent = 'Criar Projeto';
                
                // Remover ID do formulário
                form.removeAttribute('data-project-id');
                
                // Definir data de início como hoje
                document.getElementById('projectStartDate').value = new Date().toISOString().split('T')[0];
            }
            
            // Exibir modal
            modal.classList.remove('hidden');
            
            // Focar no primeiro campo
            document.getElementById('projectName').focus();
        }

        // Abrir modal de detalhe do projeto
        function openProjectDetail(projectId) {
            const project = store.projects.find(p => p.id === projectId);
            if (!project) {
                showToast('Projeto não encontrado.', 'error');
                return;
            }
            
            const modal = document.getElementById('projectDetailModal');
            const content = document.getElementById('projectDetailContent');
            const title = document.getElementById('projectDetailTitle');
            
            title.textContent = project.name;
            
            // Calcular progresso
            const projectTasks = store.tasks.filter(t => t.projectId === project.id);
            const totalTasks = projectTasks.length;
            const completedTasks = projectTasks.filter(t => t.status === 'completed').length;
            const progress = totalTasks > 0 ? Math.round((completedTasks / totalTasks) * 100) : 0;
            
            // Status baseado no progresso
            let statusBadge, statusClass;
            if (progress === 100) {
                statusBadge = 'Concluído';
                statusClass = 'bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-300';
            } else if (progress >= 70) {
                statusBadge = 'Avançado';
                statusClass = 'bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-300';
            } else if (progress >= 30) {
                statusBadge = 'Em progresso';
                statusClass = 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/30 dark:text-yellow-300';
            } else {
                statusBadge = 'Iniciando';
                statusClass = 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
            }
            
            // Prioridade
            let priorityBadge, priorityClass;
            switch (project.priority) {
                case 'high':
                    priorityBadge = 'Alta';
                    priorityClass = 'bg-red-100 text-red-800 dark:bg-red-900/30 dark:text-red-300';
                    break;
                case 'medium':
                    priorityBadge = 'Média';
                    priorityClass = 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/30 dark:text-yellow-300';
                    break;
                default:
                    priorityBadge = 'Baixa';
                    priorityClass = 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
            }
            
            // Formatar datas
            const startDate = project.startDate ? formatDate(project.startDate) : 'Não definido';
            const dueDate = project.dueDate ? formatDate(project.dueDate) : 'Não definido';
            const createdDate = formatDate(project.createdAt);
            
            // Construir conteúdo do modal
            content.innerHTML = `
                <div class="space-y-6">
                    <!-- Informações gerais -->
                    <div class="bg-gray-50 dark:bg-gray-700 rounded-lg p-4">
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div>
                                <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400">Status</h4>
                                <span class="inline-block mt-1 px-2 py-1 rounded-full text-xs ${statusClass}">
                                    ${statusBadge}
                                </span>
                            </div>
                            <div>
                                <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400">Prioridade</h4>
                                <span class="inline-block mt-1 px-2 py-1 rounded-full text-xs ${priorityClass}">
                                    ${priorityBadge}
                                </span>
                            </div>
                            <div>
                                <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400">Data de início</h4>
                                <p>${startDate}</p>
                            </div>
                            <div>
                                <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400">Data de conclusão</h4>
                                <p>${dueDate}</p>
                            </div>
                            <div>
                                <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400">Criado em</h4>
                                <p>${createdDate}</p>
                            </div>
                            <div>
                                <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400">Progresso</h4>
                                <div class="w-full bg-gray-200 dark:bg-gray-800 rounded-full h-2.5 mt-2">
                                    <div class="bg-primary-500 h-2.5 rounded-full" style="width: ${progress}%"></div>
                                </div>
                                <span class="text-xs text-gray-500 dark:text-gray-400">${progress}% (${completedTasks}/${totalTasks} tarefas)</span>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Descrição e detalhes (5W2H) -->
                    <div>
                        <h3 class="text-lg font-medium mb-2">Sobre o projeto</h3>
                        <p class="text-gray-800 dark:text-gray-200 mb-4">${project.description || 'Sem descrição'}</p>
                        
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                            <div class="bg-white dark:bg-gray-800 p-4 rounded-lg border border-gray-200 dark:border-gray-700">
                                <h4 class="font-medium flex items-center mb-2">
                                    <span class="bg-blue-100 dark:bg-blue-900/30 text-blue-800 dark:text-blue-300 w-6 h-6 rounded-full flex items-center justify-center mr-2">?</span>
                                    Por quê? (Why)
                                </h4>
                                <p>${project.why || 'Não especificado'}</p>
                            </div>
                            
                            <div class="bg-white dark:bg-gray-800 p-4 rounded-lg border border-gray-200 dark:border-gray-700">
                                <h4 class="font-medium flex items-center mb-2">
                                    <span class="bg-green-100 dark:bg-green-900/30 text-green-800 dark:text-green-300 w-6 h-6 rounded-full flex items-center justify-center mr-2">W</span>
                                    Quem? (Who)
                                </h4>
                                <p>${project.who || 'Não especificado'}</p>
                            </div>
                            
                            <div class="bg-white dark:bg-gray-800 p-4 rounded-lg border border-gray-200 dark:border-gray-700">
                                <h4 class="font-medium flex items-center mb-2">
                                    <span class="bg-purple-100 dark:bg-purple-900/30 text-purple-800 dark:text-purple-300 w-6 h-6 rounded-full flex items-center justify-center mr-2">W</span>
                                    Onde? (Where)
                                </h4>
                                <p>${project.where || 'Não especificado'}</p>
                            </div>
                            
                            <div class="bg-white dark:bg-gray-800 p-4 rounded-lg border border-gray-200 dark:border-gray-700">
                                <h4 class="font-medium flex items-center mb-2">
                                    <span class="bg-yellow-100 dark:bg-yellow-900/30 text-yellow-800 dark:text-yellow-300 w-6 h-6 rounded-full flex items-center justify-center mr-2">H</span>
                                    Como? (How)
                                </h4>
                                <p>${project.how || 'Não especificado'}</p>
                            </div>
                            
                            <div class="bg-white dark:bg-gray-800 p-4 rounded-lg border border-gray-200 dark:border-gray-700 md:col-span-2">
                                <h4 class="font-medium flex items-center mb-2">
                                    <span class="bg-red-100 dark:bg-red-900/30 text-red-800 dark:text-red-300 w-6 h-6 rounded-full flex items-center justify-center mr-2">$</span>
                                    Quanto custa? (How much)
                                </h4>
                                <p>${project.howMuch || 'Não especificado'}</p>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Tarefas do projeto -->
                    <div>
                        <div class="flex justify-between items-center mb-2">
                            <h3 class="text-lg font-medium">Tarefas</h3>
                            <button id="newTaskInProject" class="text-sm bg-primary-500 hover:bg-primary-600 text-white px-3 py-1 rounded-md" data-project-id="${project.id}">
                                <i class="fas fa-plus mr-1"></i> Nova Tarefa
                            </button>
                        </div>
                        
                        <div class="bg-white dark:bg-gray-800 rounded-lg border border-gray-200 dark:border-gray-700 overflow-hidden">
                            ${projectTasks.length > 0 ? `
                                <table class="min-w-full divide-y divide-gray-200 dark:divide-gray-700">
                                    <thead class="bg-gray-50 dark:bg-gray-700">
                                        <tr>
                                            <th scope="col" class="px-4 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Nome</th>
                                            <th scope="col" class="px-4 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Status</th>
                                            <th scope="col" class="px-4 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Prioridade</th>
                                            <th scope="col" class="px-4 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Prazo</th>
                                            <th scope="col" class="px-4 py-3 text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider text-right">Ações</th>
                                        </tr>
                                    </thead>
                                    <tbody class="divide-y divide-gray-200 dark:divide-gray-700">
                                        ${projectTasks.map(task => {
                                            // Status
                                            let statusBadge, statusClass;
                                            switch (task.status) {
                                                case 'completed':
                                                    statusBadge = 'Concluída';
                                                    statusClass = 'bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-300';
                                                    break;
                                                case 'in_progress':
                                                    statusBadge = 'Em Progresso';
                                                    statusClass = 'bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-300';
                                                    break;
                                                default:
                                                    statusBadge = 'A Fazer';
                                                    statusClass = 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
                                            }
                                            
                                            // Prioridade
                                            let priorityBadge, priorityClass;
                                            switch (task.priority) {
                                                case 'high':
                                                    priorityBadge = 'Alta';
                                                    priorityClass = 'bg-red-100 text-red-800 dark:bg-red-900/30 dark:text-red-300';
                                                    break;
                                                case 'medium':
                                                    priorityBadge = 'Média';
                                                    priorityClass = 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/30 dark:text-yellow-300';
                                                    break;
                                                default:
                                                    priorityBadge = 'Baixa';
                                                    priorityClass = 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
                                            }
                                            
                                            // Data formatada
                                            const dueDate = task.dueDate ? formatDate(task.dueDate) : 'Não definido';
                                            
                                            return `
                                                <tr>
                                                    <td class="px-4 py-3 whitespace-nowrap">
                                                        <div class="flex items-center">
                                                            <div class="text-sm font-medium">${task.name}</div>
                                                        </div>
                                                    </td>
                                                    <td class="px-4 py-3 whitespace-nowrap">
                                                        <span class="px-2 py-1 inline-flex text-xs leading-5 font-semibold rounded-full ${statusClass}">
                                                            ${statusBadge}
                                                        </span>
                                                    </td>
                                                    <td class="px-4 py-3 whitespace-nowrap">
                                                        <span class="px-2 py-1 inline-flex text-xs leading-5 font-semibold rounded-full ${priorityClass}">
                                                            ${priorityBadge}
                                                        </span>
                                                    </td>
                                                    <td class="px-4 py-3 whitespace-nowrap text-sm text-gray-500 dark:text-gray-400">
                                                        ${dueDate}
                                                    </td>
                                                    <td class="px-4 py-3 whitespace-nowrap text-right text-sm font-medium">
                                                        <button class="text-primary-500 hover:text-primary-700 dark:text-primary-400 dark:hover:text-primary-300 view-task-btn" data-id="${task.id}">
                                                            <i class="fas fa-eye"></i>
                                                        </button>
                                                        <button class="ml-3 text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300 edit-task-btn" data-id="${task.id}">
                                                            <i class="fas fa-edit"></i>
                                                        </button>
                                                    </td>
                                                </tr>
                                            `;
                                        }).join('')}
                                    </tbody>
                                </table>
                            ` : `
                                <div class="py-8 text-center">
                                    <i class="fas fa-tasks text-gray-300 dark:text-gray-600 text-4xl mb-3"></i>
                                    <p class="text-gray-500 dark:text-gray-400">Nenhuma tarefa encontrada para este projeto.</p>
                                    <button id="firstTaskBtn" class="mt-4 text-sm bg-primary-500 hover:bg-primary-600 text-white px-4 py-2 rounded-md" data-project-id="${project.id}">
                                        <i class="fas fa-plus mr-1"></i> Adicionar Primeira Tarefa
                                    </button>
                                </div>
                            `}
                        </div>
                    </div>
                    
                    <!-- Botões de ação -->
                    <div class="flex justify-end space-x-3 pt-4 border-t border-gray-200 dark:border-gray-700">
                        <button id="editProjectBtn" class="px-4 py-2 text-sm font-medium text-white bg-primary-500 hover:bg-primary-600 rounded-md shadow-sm" data-id="${project.id}">
                            <i class="fas fa-edit mr-1"></i> Editar Projeto
                        </button>
                        <button id="deleteProjectBtn" class="px-4 py-2 text-sm font-medium text-white bg-red-600 hover:bg-red-700 rounded-md shadow-sm" data-id="${project.id}">
                            <i class="fas fa-trash-alt mr-1"></i> Excluir Projeto
                        </button>
                    </div>
                </div>
            `;
            
            // Exibir modal
            modal.classList.remove('hidden');
            
            // Adicionar evento aos botões
            document.getElementById('editProjectBtn').addEventListener('click', function() {
                const projectId = this.getAttribute('data-id');
                editProject(projectId);
                modal.classList.add('hidden');
            });
            
            document.getElementById('deleteProjectBtn').addEventListener('click', function() {
                const projectId = this.getAttribute('data-id');
                modal.classList.add('hidden');
                confirmDeleteProject(projectId);
            });
            
            document.getElementById('newTaskInProject').addEventListener('click', function() {
                const projectId = this.getAttribute('data-project-id');
                openTaskModal(null, projectId);
                modal.classList.add('hidden');
            });
            
            if (document.getElementById('firstTaskBtn')) {
                document.getElementById('firstTaskBtn').addEventListener('click', function() {
                    const projectId = this.getAttribute('data-project-id');
                    openTaskModal(null, projectId);
                    modal.classList.add('hidden');
                });
            }
            
            // Adicionar eventos às tarefas
            document.querySelectorAll('.view-task-btn').forEach(btn => {
                btn.addEventListener('click', function() {
                    const taskId = this.getAttribute('data-id');
                    modal.classList.add('hidden');
                    viewTask(taskId);
                });
            });
            
            document.querySelectorAll('.edit-task-btn').forEach(btn => {
                btn.addEventListener('click', function() {
                    const taskId = this.getAttribute('data-id');
                    modal.classList.add('hidden');
                    editTask(taskId);
                });
            });
            
            // Botão de fechar
            document.getElementById('closeProjectDetail').addEventListener('click', function() {
                modal.classList.add('hidden');
            });
            
            // Fechar ao clicar no backdrop
            document.getElementById('projectDetailModalBackdrop').addEventListener('click', function() {
                modal.classList.add('hidden');
            });
        }

        // Abrir modal de tarefa (novo ou edição)
        function openTaskModal(taskData = null, presetProjectId = null) {
            const modal = document.getElementById('taskModal');
            const form = document.getElementById('taskForm');
            const title = document.getElementById('taskModalTitle');
            const submitBtn = form.querySelector('button[type="submit"]');
            const projectSelect = document.getElementById('taskProjectId');
            
            // Limpar dados anteriores
            form.reset();
            
            // Preencher o select de projetos
            projectSelect.innerHTML = '';
            store.projects.forEach(project => {
                const option = document.createElement('option');
                option.value = project.id;
                option.textContent = project.name;
                projectSelect.appendChild(option);
            });
            
            // Configurar para nova tarefa ou edição
            if (taskData) {
                title.textContent = 'Editar Tarefa';
                submitBtn.textContent = 'Atualizar Tarefa';
                
                // Preencher formulário com dados existentes
                document.getElementById('taskId').value = taskData.id;
                document.getElementById('taskName').value = taskData.name || '';
                document.getElementById('taskDescription').value = taskData.description || '';
                document.getElementById('taskProjectId').value = taskData.projectId || '';
                document.getElementById('taskStatus').value = taskData.status || 'todo';
                document.getElementById('taskPriority').value = taskData.priority || 'medium';
                document.getElementById('taskDueDate').value = taskData.dueDate || '';
                document.getElementById('taskWho').value = taskData.who || '';
                document.getElementById('taskHow').value = taskData.how || '';
            } else {
                title.textContent = 'Nova Tarefa';
                submitBtn.textContent = 'Criar Tarefa';
                document.getElementById('taskId').value = '';
                
                // Se foi passado um ID de projeto, selecionar
                if (presetProjectId) {
                    document.getElementById('taskProjectId').value = presetProjectId;
                }
            }
            
            // Exibir modal
            modal.classList.remove('hidden');
            
            // Focar no primeiro campo
            document.getElementById('taskName').focus();
            
            // Adicionar evento ao formulário
            form.onsubmit = function(e) {
                e.preventDefault();
                
                const taskId = document.getElementById('taskId').value;
                const taskData = {
                    name: document.getElementById('taskName').value,
                    description: document.getElementById('taskDescription').value,
                    projectId: document.getElementById('taskProjectId').value,
                    status: document.getElementById('taskStatus').value,
                    priority: document.getElementById('taskPriority').value,
                    dueDate: document.getElementById('taskDueDate').value,
                    who: document.getElementById('taskWho').value,
                    how: document.getElementById('taskHow').value
                };
                
                if (!taskData.name) {
                    showToast('O nome da tarefa é obrigatório.', 'warning');
                    return;
                }
                
                if (!taskData.projectId) {
                    showToast('Selecione um projeto para a tarefa.', 'warning');
                    return;
                }
                
                // Criar nova tarefa ou atualizar existente
                if (taskId) {
                    // Atualizar tarefa
                    updateTask(taskId, taskData);
                    showToast('Tarefa atualizada com sucesso!', 'success');
                } else {
                    // Criar nova tarefa
                    createTask(taskData);
                    showToast('Tarefa criada com sucesso!', 'success');
                }
                
                // Fechar modal
                modal.classList.add('hidden');
                
                // Atualizar interface
                if (currentView === 'tasks') {
                    renderTasks();
                } else if (currentView === 'kanban') {
                    renderKanban();
                } else if (currentView === 'dashboard') {
                    renderDashboard();
                } else if (currentView === 'timeline') {
                    renderTimeline();
                }
                
                updateStats();
            };
            
            // Botões de cancelar/fechar
            document.getElementById('cancelTask').addEventListener('click', function() {
                modal.classList.add('hidden');
            });
            
            document.getElementById('closeTaskModal').addEventListener('click', function() {
                modal.classList.add('hidden');
            });
            
            document.getElementById('taskModalBackdrop').addEventListener('click', function() {
                modal.classList.add('hidden');
            });
        }

        // Ver detalhes da tarefa
        function viewTask(taskId) {
            const task = store.tasks.find(t => t.id === taskId);
            if (!task) {
                showToast('Tarefa não encontrada.', 'error');
                return;
            }
            
            const project = store.projects.find(p => p.id === task.projectId);
            
            // Status da tarefa
            let statusBadge, statusClass;
            switch (task.status) {
                case 'completed':
                    statusBadge = 'Concluída';
                    statusClass = 'bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-300';
                    break;
                case 'in_progress':
                    statusBadge = 'Em Progresso';
                    statusClass = 'bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-300';
                    break;
                default:
                    statusBadge = 'A Fazer';
                    statusClass = 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
            }
            
            // Prioridade
            let priorityBadge, priorityClass;
            switch (task.priority) {
                case 'high':
                    priorityBadge = 'Alta';
                    priorityClass = 'bg-red-100 text-red-800 dark:bg-red-900/30 dark:text-red-300';
                    break;
                case 'medium':
                    priorityBadge = 'Média';
                    priorityClass = 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/30 dark:text-yellow-300';
                    break;
                default:
                    priorityBadge = 'Baixa';
                    priorityClass = 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-300';
            }
            
            // Data formatada
            const dueDate = task.dueDate ? formatDate(task.dueDate) : 'Não definido';
            const createdDate = formatDate(task.createdAt);
            
            // Criar modal de detalhes
            const modal = document.createElement('div');
            modal.className = 'fixed inset-0 z-50 overflow-y-auto flex items-center justify-center';
            modal.innerHTML = `
                <div class="fixed inset-0 bg-black bg-opacity-50 transition-opacity"></div>
                <div class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-xl transform transition-all max-w-lg w-full mx-4">
                    <div class="px-6 py-4 border-b border-gray-200 dark:border-gray-700 flex justify-between items-center">
                        <h3 class="text-lg font-semibold">${task.name}</h3>
                        <button class="close-modal text-gray-400 hover:text-gray-500 focus:outline-none">
                            <i class="fas fa-times"></i>
                        </button>
                    </div>
                    <div class="p-6 space-y-4">
                        <div class="flex flex-wrap gap-2">
                            <span class="px-2 py-1 rounded-full text-xs ${statusClass}">
                                ${statusBadge}
                            </span>
                            <span class="px-2 py-1 rounded-full text-xs ${priorityClass}">
                                Prioridade: ${priorityBadge}
                            </span>
                        </div>
                        
                        <div class="bg-gray-50 dark:bg-gray-700 rounded-lg p-4">
                            <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-1">Projeto</h4>
                            <p class="font-medium">${project ? project.name : 'Projeto não encontrado'}</p>
                        </div>
                        
                        <div>
                            <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-1">Descrição</h4>
                            <p>${task.description || 'Sem descrição'}</p>
                        </div>
                        
                        <div class="grid grid-cols-2 gap-4">
                            <div>
                                <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-1">Prazo</h4>
                                <p>${dueDate}</p>
                            </div>
                            <div>
                                <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-1">Criada em</h4>
                                <p>${createdDate}</p>
                            </div>
                        </div>
                        
                        <div>
                            <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-1">Responsável</h4>
                            <p>${task.who || 'Não especificado'}</p>
                        </div>
                        
                        <div>
                            <h4 class="text-sm font-medium text-gray-500 dark:text-gray-400 mb-1">Como fazer</h4>
                            <p>${task.how || 'Não especificado'}</p>
                        </div>
                        
                        <!-- Ações para a tarefa -->
                        <div class="pt-4 flex justify-end space-x-3 border-t border-gray-200 dark:border-gray-700">
                            <button class="change-status-btn px-4 py-2 text-sm font-medium text-white rounded-md shadow-sm ${
                                task.status === 'completed' 
                                    ? 'bg-gray-500 hover:bg-gray-600' 
                                    : 'bg-green-500 hover:bg-green-600'
                            }" data-id="${task.id}" data-status="${task.status}">
                                <i class="fas fa-${task.status === 'completed' ? 'undo' : 'check'} mr-1"></i>
                                ${task.status === 'completed' ? 'Reabrir Tarefa' : 'Marcar Concluída'}
                            </button>
                            <button class="edit-task-modal-btn px-4 py-2 text-sm font-medium text-white bg-primary-500 hover:bg-primary-600 rounded-md shadow-sm" data-id="${task.id}">
                                <i class="fas fa-edit mr-1"></i> Editar
                            </button>
                            <button class="delete-task-modal-btn px-4 py-2 text-sm font-medium text-white bg-red-600 hover:bg-red-700 rounded-md shadow-sm" data-id="${task.id}">
                                <i class="fas fa-trash-alt mr-1"></i> Excluir
                            </button>
                        </div>
                    </div>
                </div>
            `;
            
            document.body.appendChild(modal);
            
            // Adicionar eventos
            modal.querySelector('.close-modal').addEventListener('click', function() {
                document.body.removeChild(modal);
            });
            
            modal.querySelector('.change-status-btn').addEventListener('click', function() {
                const taskId = this.getAttribute('data-id');
                const currentStatus = this.getAttribute('data-status');
                
                // Alternar status
                const newStatus = currentStatus === 'completed' ? 'todo' : 'completed';
                
                updateTask(taskId, { status: newStatus });
                document.body.removeChild(modal);
                
                // Atualizar interface se necessário
                if (currentView === 'tasks') {
                    renderTasks();
                } else if (currentView === 'kanban') {
                    renderKanban();
                } else if (currentView === 'dashboard') {
                    renderDashboard();
                } else if (currentView === 'timeline') {
                    renderTimeline();
                }
                
                updateStats();
                
                showToast(
                    newStatus === 'completed' ? 
                    'Tarefa marcada como concluída!' : 
                    'Tarefa reaberta!', 
                    'success'
                );
            });
            
            modal.querySelector('.edit-task-modal-btn').addEventListener('click', function() {
                const taskId = this.getAttribute('data-id');
                document.body.removeChild(modal);
                editTask(taskId);
            });
            
            modal.querySelector('.delete-task-modal-btn').addEventListener('click', function() {
                const taskId = this.getAttribute('data-id');
                document.body.removeChild(modal);
                confirmDeleteTask(taskId);
            });
            
            // Fechar ao clicar no backdrop
            modal.addEventListener('click', function(e) {
                if (e.target === modal) {
                    document.body.removeChild(modal);
                }
            });
        }

        // Abrir modal de ideia
        function openIdeaModal(ideaData = null) {
            const modal = document.getElementById('ideaModal');
            const form = document.getElementById('ideaForm');
            const title = document.getElementById('ideaModalTitle');
            const submitBtn = form.querySelector('button[type="submit"]');
            
            // Limpar dados anteriores
            form.reset();
            
            // Configurar para nova ideia ou edição
            if (ideaData) {
                title.textContent = 'Editar Ideia';
                submitBtn.textContent = 'Atualizar Ideia';
                
                // Preencher formulário com dados existentes
                document.getElementById('ideaId').value = ideaData.id;
                document.getElementById('ideaTitle').value = ideaData.title || '';
                document.getElementById('ideaDescription').value = ideaData.description || '';
                document.getElementById('ideaCategory').value = ideaData.category || 'feature';
                document.getElementById('ideaTags').value = ideaData.tags ? ideaData.tags.join(', ') : '';
            } else {
                title.textContent = 'Nova Ideia';
                submitBtn.textContent = 'Registrar Ideia';
                document.getElementById('ideaId').value = '';
            }
            
            // Exibir modal
            modal.classList.remove('hidden');
            
            // Focar no primeiro campo
            document.getElementById('ideaTitle').focus();
            
            // Adicionar evento ao formulário
            form.onsubmit = function(e) {
                e.preventDefault();
                
                const ideaId = document.getElementById('ideaId').value;
                
                // Processar tags
                const tagsInput = document.getElementById('ideaTags').value;
                const tags = tagsInput ? 
                    tagsInput.split(',').map(tag => tag.trim()).filter(tag => tag) : 
                    [];
                
                const ideaData = {
                    title: document.getElementById('ideaTitle').value,
                    description: document.getElementById('ideaDescription').value,
                    category: document.getElementById('ideaCategory').value,
                    tags
                };
                
                if (!ideaData.title) {
                    showToast('O título da ideia é obrigatório.', 'warning');
                    return;
                }
                
                if (!ideaData.description) {
                    showToast('A descrição da ideia é obrigatória.', 'warning');
                    return;
                }
                
                // Criar nova ideia ou atualizar existente
                if (ideaId) {
                    // Atualizar ideia
                    updateIdea(ideaId, ideaData);
                    showToast('Ideia atualizada com sucesso!', 'success');
                } else {
                    // Criar nova ideia
                    createIdea(ideaData);
                    showToast('Ideia registrada com sucesso!', 'success');
                }
                
                // Fechar modal
                modal.classList.add('hidden');
                
                // Atualizar interface
                if (currentView === 'ideas') {
                    renderIdeas();
                } else if (currentView === 'dashboard') {
                    renderDashboard();
                }
                
                updateStats();
            };
            
            // Botões de cancelar/fechar
            document.getElementById('cancelIdea').addEventListener('click', function() {
                modal.classList.add('hidden');
            });
            
            document.getElementById('closeIdeaModal').addEventListener('click', function() {
                modal.classList.add('hidden');
            });
            
            document.getElementById('ideaModalBackdrop').addEventListener('click', function() {
                modal.classList.add('hidden');
            });
        }

        // Editar projeto
        function editProject(projectId) {
            const project = store.projects.find(p => p.id === projectId);
            if (!project) {
                showToast('Projeto não encontrado.', 'error');
                return;
            }
            
            openNewProjectModal(project);
        }

        // Editar tarefa
        function editTask(taskId) {
            const task = store.tasks.find(t => t.id === taskId);
            if (!task) {
                showToast('Tarefa não encontrada.', 'error');
                return;
            }
            
            openTaskModal(task);
        }

        // Editar ideia
        function editIdea(ideaId) {
            const idea = store.ideas.find(i => i.id === ideaId);
            if (!idea) {
                showToast('Ideia não encontrada.', 'error');
                return;
            }
            
            openIdeaModal(idea);
        }

        // Confirmação de exclusão de projeto
        function confirmDeleteProject(projectId) {
            const project = store.projects.find(p => p.id === projectId);
            if (!project) {
                showToast('Projeto não encontrado.', 'error');
                return;
            }
            
            const modal = document.getElementById('confirmDeleteModal');
            const message = document.getElementById('confirmDeleteMessage');
            
            // Contar tarefas do projeto
            const projectTasks = store.tasks.filter(t => t.projectId === projectId);
            
            message.innerHTML = `
                Tem certeza que deseja excluir o projeto <strong>${project.name}</strong>?
                <br><br>
                Esta ação também excluirá <strong>${projectTasks.length}</strong> tarefas associadas.
                <br><br>
                Esta ação não pode ser desfeita.
            `;
            
            currentItemForDeletion = { type: 'project', id: projectId };
            
            modal.classList.remove('hidden');
        }

        // Confirmação de exclusão de tarefa
        function confirmDeleteTask(taskId) {
            const task = store.tasks.find(t => t.id === taskId);
            if (!task) {
                showToast('Tarefa não encontrada.', 'error');
                return;
            }
            
            const modal = document.getElementById('confirmDeleteModal');
            const message = document.getElementById('confirmDeleteMessage');
            
            message.innerHTML = `
                Tem certeza que deseja excluir a tarefa <strong>${task.name}</strong>?
                <br><br>
                Esta ação não pode ser desfeita.
            `;
            
            currentItemForDeletion = { type: 'task', id: taskId };
            
            modal.classList.remove('hidden');
        }

        // Confirmação de exclusão de ideia
        function confirmDeleteIdea(ideaId) {
            const idea = store.ideas.find(i => i.id === ideaId);
            if (!idea) {
                showToast('Ideia não encontrada.', 'error');
                return;
            }
            
            const modal = document.getElementById('confirmDeleteModal');
            const message = document.getElementById('confirmDeleteMessage');
            
            message.innerHTML = `
                Tem certeza que deseja excluir a ideia <strong>${idea.title}</strong>?
                <br><br>
                Esta ação não pode ser desfeita.
            `;
            
            currentItemForDeletion = { type: 'idea', id: ideaId };
            
            modal.classList.remove('hidden');
        }

        // Navegar para uma visualização
        function navigateTo(view) {
            // Verificar se a visualização é válida
            const validViews = ['dashboard', 'projects', 'tasks', 'kanban', 'timeline', 'logs', 'ideas'];
            if (!validViews.includes(view)) {
                view = 'dashboard';
            }
            
            // Atualizar visualização atual
            currentView = view;
            
            // Remover classe ativa de todos os links
            document.querySelectorAll('.nav-link').forEach(link => {
                link.classList.remove('text-primary-500', 'bg-primary-50', 'dark:bg-primary-900/30');
                link.classList.add('hover:bg-gray-100', 'dark:hover:bg-gray-700');
            });
            
            // Adicionar classe ativa ao link correspondente
            const activeLink = document.querySelector(`.nav-link[href="#${view}"]`);
            if (activeLink) {
                activeLink.classList.add('text-primary-500', 'bg-primary-50', 'dark:bg-primary-900/30');
                activeLink.classList.remove('hover:bg-gray-100', 'dark:hover:bg-gray-700');
            }
            
            // Renderizar a visualização correspondente
            switch (view) {
                case 'dashboard':
                    renderDashboard();
                    break;
                case 'projects':
                    renderProjects();
                    break;
                case 'tasks':
                    renderTasks();
                    break;
                case 'kanban':
                    renderKanban();
                    break;
                case 'timeline':
                    renderTimeline();
                    break;
                case 'logs':
                    renderLogs();
                    break;
                case 'ideas':
                    renderIdeas();
                    break;
            }
        }

        // Obter endereço atual e navegar para a visualização correspondente
        function handleHashChange() {
            const hash = window.location.hash.substring(1) || 'dashboard';
            navigateTo(hash);
        }

        // Inicialização quando o DOM estiver pronto
        document.addEventListener('DOMContentLoaded', function() {
            console.log('DOM carregado, iniciando aplicação...');
            
            // Esconder o overlay de carregamento
            const loadingOverlay = document.getElementById('loadingOverlay');
            loadingOverlay.classList.add('hidden');
            
            try {
                // Tentar carregar dados existentes
                const dataLoaded = loadStore();
                
                // Se não houver dados, inicializar com dados de exemplo
                if (!dataLoaded || store.projects.length === 0) {
                    console.log('Nenhum dado encontrado. Inicializando dados de exemplo...');
                    initializeExampleData();
                }
                
                // Atualizar interface
                updateStats();
                updateStorageIndicator();
                
                // Processar hash inicial
                handleHashChange();
                
                // Adicionar evento para mudanças de hash
                window.addEventListener('hashchange', handleHashChange);
                
                console.log('Aplicação inicializada com sucesso');
                showToast('Bem-vindo ao ProjetoPRO!', 'success');
            } catch (error) {
                console.error('Erro ao inicializar aplicação:', error);
                showToast('Erro ao carregar dados. Verifique o console para detalhes.', 'error');
            }
            
            // GLOBAL EVENT LISTENERS
            
            // Toggle do modo escuro
            document.getElementById('toggleDarkMode').addEventListener('click', function() {
                document.documentElement.classList.toggle('dark');
                
                // Fechar menu
                document.getElementById('userMenu').classList.add('hidden');
            });
            
            // Toggle do menu de usuário
            document.getElementById('userMenuBtn').addEventListener('click', function() {
                document.getElementById('userMenu').classList.toggle('hidden');
            });
            
            // Fechar menu quando clicar fora
            document.addEventListener('click', function(e) {
                const userMenu = document.getElementById('userMenu');
                const userMenuBtn = document.getElementById('userMenuBtn');
                
                if (userMenu && !userMenu.classList.contains('hidden') && 
                    !userMenuBtn.contains(e.target) && !userMenu.contains(e.target)) {
                    userMenu.classList.add('hidden');
                }
            });
            
            // Abrir modal de configurações
            document.getElementById('settingsBtn').addEventListener('click', function(e) {
                e.preventDefault();
                
                // Fechar menu
                document.getElementById('userMenu').classList.add('hidden');
                
                // Abrir modal de configurações
                document.getElementById('settingsModal').classList.remove('hidden');
                
                // Configurar toggle de modo escuro
                document.getElementById('darkModeToggle').checked = document.documentElement.classList.contains('dark');
            });
            
            // Fechar modal de configurações
            document.getElementById('closeSettingsModal').addEventListener('click', function() {
                document.getElementById('settingsModal').classList.add('hidden');
            });
            
            document.getElementById('settingsModalBackdrop').addEventListener('click', function() {
                document.getElementById('settingsModal').classList.add('hidden');
            });
            
            // Toggle de modo escuro nas configurações
            document.getElementById('darkModeToggle').addEventListener('change', function() {
                if (this.checked) {
                    document.documentElement.classList.add('dark');
                } else {
                    document.documentElement.classList.remove('dark');
                }
            });
            
            // Modal de confirmação de exclusão
            document.getElementById('confirmDelete').addEventListener('click', function() {
                if (!currentItemForDeletion) return;
                
                const { type, id } = currentItemForDeletion;
                
                // Excluir item conforme tipo
                let success = false;
                let message = '';
                
                switch (type) {
                    case 'project':
                        success = deleteProject(id);
                        message = 'Projeto excluído com sucesso!';
                        break;
                    case 'task':
                        success = deleteTask(id);
                        message = 'Tarefa excluída com sucesso!';
                        break;
                    case 'idea':
                        success = deleteIdea(id);
                        message = 'Ideia excluída com sucesso!';
                        break;
                }
                
                // Fechar modal
                document.getElementById('confirmDeleteModal').classList.add('hidden');
                
                // Limpar referência
                currentItemForDeletion = null;
                
                if (success) {
                    showToast(message, 'success');
                    
                    // Atualizar interface
                    switch (currentView) {
                        case 'dashboard':
                            renderDashboard();
                            break;
                        case 'projects':
                            renderProjects();
                            break;
                        case 'tasks':
                            renderTasks();
                            break;
                        case 'kanban':
                            renderKanban();
                            break;
                        case 'timeline':
                            renderTimeline();
                            break;
                        case 'ideas':
                            renderIdeas();
                            break;
                    }
                    
                    updateStats();
                } else {
                    showToast('Erro ao excluir o item.', 'error');
                }
            });
            
            document.getElementById('cancelDelete').addEventListener('click', function() {
                document.getElementById('confirmDeleteModal').classList.add('hidden');
                currentItemForDeletion = null;
            });
            
            document.getElementById('confirmDeleteModalBackdrop').addEventListener('click', function() {
                document.getElementById('confirmDeleteModal').classList.add('hidden');
                currentItemForDeletion = null;
            });
            
            // Importação e exportação
            document.getElementById('importBtn').addEventListener('click', function() {
                document.getElementById('dataTransferModal').classList.remove('hidden');
                document.getElementById('importContent').style.display = 'block';
                document.getElementById('dataTransferModalTitle').textContent = 'Importar Dados';
            });
            
            document.getElementById('exportBtn').addEventListener('click', function() {
                document.getElementById('dataTransferModal').classList.remove('hidden');
                document.getElementById('importContent').style.display = 'none';
                document.getElementById('dataTransferModalTitle').textContent = 'Exportar Dados';
            });
            
            document.getElementById('closeDataTransferModal').addEventListener('click', function() {
                document.getElementById('dataTransferModal').classList.add('hidden');
            });
            
            document.getElementById('dataTransferModalBackdrop').addEventListener('click', function() {
                document.getElementById('dataTransferModal').classList.add('hidden');
            });
            
            document.getElementById('importDataBtn').addEventListener('click', function() {
                const fileInput = document.getElementById('importFile');
                if (!fileInput.files || fileInput.files.length === 0) {
                    showToast('Selecione um arquivo para importar.', 'warning');
                    return;
                }
                
                const file = fileInput.files[0];
                const reader = new FileReader();
                
                reader.onload = function(e) {
                    try {
                        const importedData = JSON.parse(e.target.result);
                        
                        // Verificar se os dados são válidos
                        if (!importedData.projects || !Array.isArray(importedData.projects) ||
                            !importedData.tasks || !Array.isArray(importedData.tasks)) {
                            showToast('O arquivo não contém dados válidos.', 'error');
                            return;
                        }
                        
                        // Confirmar substituição de dados
                        if (confirm("Esta ação substituirá todos os dados existentes. Deseja continuar?")) {
                            // Atualizar o store com os dados importados
                            store.projects = importedData.projects || [];
                            store.tasks = importedData.tasks || [];
                            store.logs = importedData.logs || [];
                            store.ideas = importedData.ideas || [];
                            
                            // Salvar dados
                            saveStore();
                            
                            // Atualizar interface
                            updateStats();
                            updateStorageIndicator();
                            handleHashChange();
                            
                            // Fechar modal
                            document.getElementById('dataTransferModal').classList.add('hidden');
                            
                            showToast('Dados importados com sucesso!', 'success');
                            
                            // Adicionar log
                            addLog({
                                type: 'import',
                                message: 'Dados importados de arquivo externo'
                            });
                        }
                    } catch (error) {
                        console.error('Erro ao importar dados:', error);
                        showToast('Erro ao importar dados. Verifique o formato do arquivo.', 'error');
                    }
                };
                
                reader.onerror = function() {
                    showToast('Erro ao ler o arquivo.', 'error');
                };
                
                reader.readAsText(file);
            });
            
            document.getElementById('exportDataBtn').addEventListener('click', function() {
                try {
                    // Preparar dados para exportação
                    const exportData = {
                        projects: store.projects,
                        tasks: store.tasks,
                        logs: store.logs,
                        ideas: store.ideas,
                        exportDate: new Date().toISOString(),
                        exportVersion: '1.0'
                    };
                    
                    // Converter para JSON
                    const jsonData = JSON.stringify(exportData, null, 2);
                    
                    // Criar blob e link de download
                    const blob = new Blob([jsonData], { type: 'application/json' });
                    const url = URL.createObjectURL(blob);
                    
                    // Criar e simular clique no link de download
                    const link = document.createElement('a');
                    link.href = url;
                    link.download = `projetoPRO_export_${new Date().toISOString().slice(0, 10)}.json`;
                    document.body.appendChild(link);
                    link.click();
                    document.body.removeChild(link);
                    
                    // Fechar modal
                    document.getElementById('dataTransferModal').classList.add('hidden');
                    
                    showToast('Dados exportados com sucesso!', 'success');
                    
                    // Adicionar log
                    addLog({
                        type: 'export',
                        message: 'Dados exportados para arquivo'
                    });
                } catch (error) {
                    console.error('Erro ao exportar dados:', error);
                    showToast('Erro ao exportar dados.', 'error');
                }
            });
            
            // Limpar todos os dados nas configurações
            document.getElementById('clearDataBtn').addEventListener('click', function() {
                if (confirm("ATENÇÃO: Esta ação apagará TODOS os seus dados. Essa ação não pode ser desfeita. Deseja realmente continuar?")) {
                    if (confirm("Tem certeza? Todos os seus projetos, tarefas, registros e ideias serão removidos permanentemente.")) {
                        // Limpar dados
                        store = {
                            projects: [],
                            tasks: [],
                            logs: [],
                            ideas: []
                        };
                        
                        // Salvar dados
                        saveStore();
                        
                        // Atualizar interface
                        updateStats();
                        updateStorageIndicator();
                        navigateTo('dashboard');
                        
                        // Fechar modal
                        document.getElementById('settingsModal').classList.add('hidden');
                        
                        showToast('Todos os dados foram removidos com sucesso.', 'success');
                    }
                }
            });
            
            // Eventos do formulário de novo projeto
            document.getElementById('newProjectForm').addEventListener('submit', function(e) {
                e.preventDefault();
                
                const projectId = this.getAttribute('data-project-id');
                const projectData = {
                    name: document.getElementById('projectName').value,
                    description: document.getElementById('projectDescription').value,
                    why: document.getElementById('projectWhy').value,
                    where: document.getElementById('projectWhere').value,
                    who: document.getElementById('projectWho').value,
                    how: document.getElementById('projectHow').value,
                    howMuch: document.getElementById('projectHowMuch').value,
                    startDate: document.getElementById('projectStartDate').value,
                    dueDate: document.getElementById('projectDueDate').value,
                    priority: document.getElementById('projectPriority').value
                };
                
                if (!projectData.name) {
                    showToast('O nome do projeto é obrigatório.', 'warning');
                    return;
                }
                
                // Validar datas
                if (projectData.startDate && projectData.dueDate) {
                    const startDate = new Date(projectData.startDate);
                    const dueDate = new Date(projectData.dueDate);
                    
                    if (dueDate < startDate) {
                        showToast('A data de conclusão não pode ser anterior à data de início.', 'warning');
                        return;
                    }
                }
                
                // Criar novo projeto ou atualizar existente
                if (projectId) {
                    // Atualizar projeto
                    updateProject(projectId, projectData);
                    showToast('Projeto atualizado com sucesso!', 'success');
                } else {
                    // Criar novo projeto
                    createProject(projectData);
                    showToast('Projeto criado com sucesso!', 'success');
                }
                
                // Fechar modal
                document.getElementById('newProjectModal').classList.add('hidden');
                
                // Atualizar interface
                updateStats();
                if (currentView === 'projects') {
                    renderProjects();
                } else if (currentView === 'dashboard') {
                    renderDashboard();
                } else if (currentView === 'timeline') {
                    renderTimeline();
                }
            });
            
            document.getElementById('cancelNewProject').addEventListener('click', function() {
                document.getElementById('newProjectModal').classList.add('hidden');
            });
            
            document.getElementById('newProjectModalBackdrop').addEventListener('click', function() {
                document.getElementById('newProjectModal').classList.add('hidden');
            });
        });
    </script>
</body>
</html>
