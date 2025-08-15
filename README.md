<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>运动任务抽奖发布器</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    
    <!-- 配置Tailwind自定义颜色和字体 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#FF6B35',
                        secondary: '#175676',
                        accent: '#4CC9F0',
                        dark: '#1A1A2E',
                        light: '#F7F7F9',
                        success: '#4CAF50',
                        warning: '#FFC107',
                        danger: '#F44336',
                        coin: '#FFD700', // 金币颜色
                        purple: '#9C27B0', // 紫色，用于污染指数
                        cat: '#FF9800', // 温柔猫猫颜色，橙色
                        badcat: '#E91E63', // 坏蛋猫猫颜色，粉红色
                        cunningcat: '#8BC34A' // 狡猾猫猫颜色，绿色
                    },
                    fontFamily: {
                        inter: ['Inter', 'system-ui', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    
    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .text-shadow {
                text-shadow: 0 2px 4px rgba(0,0,0,0.1);
            }
            .rotate-spin-slow {
                animation: spin 8s linear infinite;
            }
            .rotate-spin-fast {
                animation: spin 2s cubic-bezier(0.17, 0.67, 0.83, 0.67) infinite;
            }
            .pulse-soft {
                animation: pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite;
            }
            .prize-option-hover {
                transition: all 0.3s ease;
            }
            .prize-option-hover:hover {
                transform: translateY(-5px);
                box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            }
            .beat-box {
                width: 50px;
                height: 50px;
                margin: 0 auto;
                border-radius: 8px;
            }
            /* 通关庆祝动画 */
            .confetti {
                position: fixed;
                width: 10px;
                height: 10px;
                background-color: #f00;
                opacity: 0;
                animation: confetti-fall 5s ease-in-out forwards;
                z-index: 1000;
            }
            @keyframes confetti-fall {
                0% {
                    transform: translateY(-100vh) rotate(0deg);
                    opacity: 1;
                }
                100% {
                    transform: translateY(100vh) rotate(720deg);
                    opacity: 0;
                }
            }
        }
    </style>
</head>
<body class="font-inter bg-gradient-to-br from-light to-gray-100 min-h-screen text-dark">
    <!-- 顶部导航栏 -->
    <header class="bg-white shadow-md fixed w-full z-50 transition-all duration-300" id="main-header">
        <div class="container mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <i class="fa fa-trophy text-primary text-2xl"></i>
                <h1 class="text-xl md:text-2xl font-bold text-dark">运动任务抽奖发布器</h1>
            </div>
            <div class="flex items-center space-x-6">
                <!-- 任务计数器 -->
                <div class="hidden md:flex items-center space-x-4">
                    <div class="bg-primary/10 text-primary px-3 py-1 rounded-full text-sm flex items-center">
                        <i class="fa fa-check-circle mr-1"></i>
                        <span>完成任务: <span id="completed-tasks-count">0</span></span>
                    </div>
                    <div class="bg-secondary/10 text-secondary px-3 py-1 rounded-full text-sm flex items-center">
                        <i class="fa fa-cubes mr-1"></i>
                        <span>完成组数: <span id="completed-groups-count">0</span></span>
                    </div>
                    <!-- 金币数量 -->
                    <div class="bg-coin/10 text-coin px-3 py-1 rounded-full text-sm flex items-center">
                        <i class="fa fa-diamond mr-1"></i>
                        <span>金币: <span id="coins-count">0</span></span>
                    </div>
                    <!-- 保护卡数量 -->
                    <div class="bg-warning/10 text-warning px-3 py-1 rounded-full text-sm flex items-center">
                        <i class="fa fa-shield mr-1"></i>
                        <span>保护卡: <span id="protection-cards-count">0</span></span>
                    </div>
                    <!-- 飞镖卡状态 -->
                    <div class="bg-accent/10 text-accent px-3 py-1 rounded-full text-sm flex items-center" id="dart-card-indicator">
                        <i class="fa fa-crosshairs mr-1"></i>
                        <span>飞镖卡: <span id="dart-card-status">未启用</span></span>
                    </div>
                </div>
                <div class="flex items-center space-x-4">
                    <!-- 商店按钮 -->
                    <button id="shop-btn" class="p-2 rounded-full hover:bg-gray-100 transition-colors">
                        <i class="fa fa-shopping-cart text-secondary"></i>
                    </button>
                    <button id="settings-btn" class="p-2 rounded-full hover:bg-gray-100 transition-colors">
                        <i class="fa fa-cog text-secondary"></i>
                    </button>
                    <button id="help-btn" class="p-2 rounded-full hover:bg-gray-100 transition-colors">
                        <i class="fa fa-question-circle text-secondary"></i>
                    </button>
                </div>
            </div>
        </div>
    </header>

    <!-- 主内容区 -->
    <main class="container mx-auto px-4 pt-24 pb-16">
        <!-- 当前奖池信息和计数器(移动端) -->
        <div class="mb-8">
            <div class="text-center mb-4">
                <div class="inline-flex items-center bg-white rounded-full px-6 py-2 shadow-md">
                    <span class="text-gray-600 mr-2">当前阶段:</span>
                    <span id="current-pool" class="font-bold text-xl text-primary">阶段 1</span>
                    <div class="w-3 h-3 rounded-full bg-success ml-3" id="pool-status-indicator"></div>
                    <span id="pool-status-text" class="ml-2 text-sm text-success font-medium">可抽取</span>
                </div>
            </div>
            
            <!-- 移动端计数器 -->
            <div class="md:hidden flex flex-wrap justify-center gap-3 mb-4">
                <div class="bg-primary/10 text-primary px-3 py-1 rounded-full text-sm flex items-center">
                    <i class="fa fa-check-circle mr-1"></i>
                    <span>任务: <span id="mobile-completed-tasks-count">0</span></span>
                </div>
                <div class="bg-secondary/10 text-secondary px-3 py-1 rounded-full text-sm flex items-center">
                    <i class="fa fa-cubes mr-1"></i>
                    <span>组数: <span id="mobile-completed-groups-count">0</span></span>
                </div>
                <!-- 移动端金币数量 -->
                <div class="bg-coin/10 text-coin px-3 py-1 rounded-full text-sm flex items-center">
                    <i class="fa fa-diamond mr-1"></i>
                    <span>金币: <span id="mobile-coins-count">0</span></span>
                </div>
                <!-- 移动端保护卡数量 -->
                <div class="bg-warning/10 text-warning px-3 py-1 rounded-full text-sm flex items-center">
                    <i class="fa fa-shield mr-1"></i>
                    <span>保护卡: <span id="mobile-protection-cards-count">0</span></span>
                </div>
                <!-- 移动端飞镖卡状态 -->
                <div class="bg-accent/10 text-accent px-3 py-1 rounded-full text-sm flex items-center">
                    <i class="fa fa-crosshairs mr-1"></i>
                    <span>飞镖卡: <span id="mobile-dart-card-status">未启用</span></span>
                </div>
            </div>
        </div>
        
        <!-- 抽奖区域 -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8 mb-10">
            <!-- 左侧：奖池选项 -->
            <div class="lg:col-span-2">
                <div class="bg-white rounded-2xl shadow-lg p-6 mb-6">
                    <div class="flex justify-between items-center mb-6">
                        <h2 class="text-xl font-bold text-secondary flex items-center">
                            <i class="fa fa-list-ul mr-2"></i>运动任务
                        </h2>
                        <button id="edit-pool-btn" class="text-sm bg-secondary/10 hover:bg-secondary/20 text-secondary px-4 py-2 rounded-full transition-colors flex items-center">
                            <i class="fa fa-pencil mr-1"></i> 编辑任务
                        </button>
                    </div>
                    
                    <div id="prize-options" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-4">
                        <!-- 奖池选项将通过JS动态生成 -->
                    </div>
                </div>
                
                <!-- 抽取结果 -->
                <div class="bg-white rounded-2xl shadow-lg p-6">
                    <h2 class="text-xl font-bold text-secondary mb-4 flex items-center">
                        <i class="fa fa-gift mr-2"></i>抽取结果
                    </h2>
                    <div id="draw-result" class="min-h-[120px] flex items-center justify-center border-2 border-dashed border-gray-200 rounded-xl p-4 text-center">
                        <p class="text-gray-400">点击上方任务选项进行抽取</p>
                    </div>
                    
                    <div class="mt-4 flex justify-center space-x-3">
                        <button id="complete-task-btn" class="bg-primary hover:bg-primary/90 text-white px-6 py-3 rounded-full font-medium shadow-md hover:shadow-lg transition-all disabled:opacity-50 disabled:cursor-not-allowed" disabled>
                            <i class="fa fa-check mr-2"></i>标记为已完成
                        </button>
                        <!-- 猫猫相关按钮，默认隐藏 -->
                        <button id="attack-monster-btn" class="hidden text-white px-6 py-3 rounded-full font-medium shadow-md hover:shadow-lg transition-all">
                            <i class="fa fa-bolt mr-2"></i>挑战猫猫
                        </button>
                        <button id="escape-monster-btn" class="hidden bg-warning hover:bg-warning/90 text-white px-6 py-3 rounded-full font-medium shadow-md hover:shadow-lg transition-all">
                            <i class="fa fa-arrow-right mr-2"></i>逃避
                        </button>
                    </div>
                </div>
            </div>
            
            <!-- 右侧：星桶和奖池导航 -->
            <div class="space-y-6">
                <!-- 星桶附加任务 -->
                <div class="bg-white rounded-2xl shadow-lg p-6">
                    <div class="flex justify-between items-center mb-4">
                        <h2 class="text-xl font-bold text-secondary flex items-center">
                            <i class="fa fa-star text-warning mr-2"></i>额外挑战
                        </h2>
                        <!-- 污染指数显示 -->
                        <div class="bg-purple/10 text-purple px-3 py-1 rounded-full text-sm flex items-center">
                            <i class="fa fa-cloud mr-1"></i>
                            <span>污染指数: <span id="pollution-index">20%</span></span>
                        </div>
                    </div>
                    
                    <div class="relative">
                        <div class="w-full aspect-square max-w-[200px] mx-auto relative">
                            <div class="absolute inset-0 rounded-full bg-gradient-to-br from-warning/20 to-warning/5 border-4 border-warning/30 flex items-center justify-center">
                                <i class="fa fa-star text-5xl animate-pulse" id="star-icon"></i>
                            </div>
                            <div class="absolute -top-2 -right-2 bg-primary text-white rounded-full w-8 h-8 flex items-center justify-center text-sm font-bold shadow-md">
                                <span id="star-tasks-count">10</span>
                            </div>
                        </div>
                        
                        <div id="star-task-result" class="mt-4 min-h-[80px] border-2 border-dashed border-gray-200 rounded-xl p-3 text-center">
                            <p class="text-gray-400">抽取主任务后将获得额外挑战</p>
                        </div>
                    </div>
                </div>
                
                <!-- 奖池导航 -->
                <div class="bg-white rounded-2xl shadow-lg p-6">
                    <h2 class="text-xl font-bold text-secondary mb-4 flex items-center">
                        <i class="fa fa-th mr-2"></i>阶段导航
                    </h2>
                    
                    <div class="grid grid-cols-5 gap-2">
                        <button class="pool-nav-btn bg-primary text-white rounded-lg p-3 font-bold transition-all" data-pool="1">1</button>
                        <button class="pool-nav-btn bg-gray-200 hover:bg-gray-300 text-gray-700 rounded-lg p-3 font-bold transition-all" data-pool="2">2</button>
                        <button class="pool-nav-btn bg-gray-200 hover:bg-gray-300 text-gray-700 rounded-lg p-3 font-bold transition-all" data-pool="3">3</button>
                        <button class="pool-nav-btn bg-gray-200 hover:bg-gray-300 text-gray-700 rounded-lg p-3 font-bold transition-all" data-pool="4">4</button>
                        <button class="pool-nav-btn bg-gray-200 hover:bg-gray-300 text-gray-700 rounded-lg p-3 font-bold transition-all" data-pool="5">5</button>
                    </div>
                    
                    <div class="mt-6">
                        <button id="draw-random-btn" class="w-full bg-accent hover:bg-accent/90 text-white py-3 rounded-xl font-medium shadow-md hover:shadow-lg transition-all disabled:opacity-50 disabled:cursor-not-allowed">
                            <i class="fa fa-random mr-2"></i>随机抽取一个任务
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </main>

    <!-- 节拍器挑战模态框 -->
    <div id="beat-challenge-modal" class="fixed inset-0 bg-black/70 z-50 flex items-center justify-center hidden">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-md p-8 text-center">
            <div class="mb-6">
                <h3 class="text-2xl font-bold flex items-center justify-center" id="challenge-title">
                    <i class="fa fa-paw mr-2"></i>温柔猫猫挑战
                </h3>
                <p class="text-gray-600 mt-2">跟随节拍完成动作，坚持<span id="challenge-total-time">20</span>秒！</p>
                <div class="mt-2 text-sm font-medium" id="challenge-bpm">
                    节拍频率: <span id="current-bpm">45</span> BPM
                </div>
                <div id="time-increase-notice" class="mt-2 text-sm text-warning hidden">
                    <i class="fa fa-clock-o mr-1"></i>挑战时间增加！
                </div>
                <div id="bpm-increase-notice" class="mt-2 text-sm text-cunningcat hidden">
                    <i class="fa fa-tachometer mr-1"></i>速度每4秒增加15BPM！
                </div>
                <!-- 喵喵卡效果提示 -->
                <div id="meow-card-effect" class="mt-2 text-sm text-success hidden">
                    <i class="fa fa-bolt mr-1"></i>喵喵卡生效：挑战时间减少5秒！
                </div>
            </div>
            
            <div class="mb-6">
                <div class="h-40 flex items-center justify-center relative">
                    <div class="absolute inset-x-0 h-1 bg-gray-200 rounded-full"></div>
                    <div class="beat-box" id="beat-indicator"></div>
                </div>
                <div class="mt-4 text-xl font-bold" id="challenge-timer">20</div>
                <p class="text-sm text-gray-500">剩余秒数</p>
            </div>
            
            <div id="challenge-result" class="hidden mb-6">
                <div class="p-4 rounded-lg bg-success/10 text-success">
                    <i class="fa fa-check-circle text-2xl mb-2"></i>
                    <p class="font-medium">挑战成功！</p>
                    <p class="text-sm mt-1">猫猫已被安抚</p>
                </div>
            </div>
            
            <button id="close-challenge-modal" class="w-full bg-primary hover:bg-primary/90 text-white py-3 rounded-xl font-medium transition-colors disabled:opacity-50 disabled:cursor-not-allowed" disabled>
                挑战进行中...
            </button>
        </div>
    </div>

    <!-- 道具商店模态框 -->
    <div id="shop-modal" class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center hidden">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-2xl max-h-[90vh] overflow-y-auto">
            <div class="p-6 border-b">
                <div class="flex justify-between items-center">
                    <h3 class="text-xl font-bold text-secondary">道具商店</h3>
                    <button id="close-shop-modal" class="text-gray-500 hover:text-gray-700">
                        <i class="fa fa-times text-xl"></i>
                    </button>
                </div>
                <p class="text-gray-500 mt-1">当前拥有: <span id="shop-coins-count" class="font-medium text-coin">0</span> 金币</p>
            </div>
            
            <div class="p-6">
                <div class="space-y-6">
                    <!-- 保护卡商品 -->
                    <div class="border border-gray-200 rounded-xl p-5 hover:shadow-md transition-shadow">
                        <div class="flex items-start">
                            <div class="w-12 h-12 bg-warning/20 text-warning rounded-lg flex items-center justify-center flex-shrink-0">
                                <i class="fa fa-shield text-2xl"></i>
                            </div>
                            <div class="ml-4 flex-grow">
                                <div class="flex justify-between items-start">
                                    <h4 class="font-bold text-lg">保护卡</h4>
                                    <span class="bg-coin/10 text-coin px-2 py-1 rounded text-sm font-medium">
                                        <i class="fa fa-diamond mr-1"></i>10金币
                                    </span>
                                </div>
                                <p class="text-gray-600 text-sm mt-2">
                                    自动使用，当抽取到会跳转到之前阶段的任务时，保护卡数量减1，
                                    同时不会跳转到之前阶段。当通过十号任务跳转阶段时，所有保护卡将会清零。
                                </p>
                                <div class="mt-3 flex justify-between items-center">
                                    <span class="text-sm text-gray-500">
                                        已拥有: <span id="shop-protection-count" class="font-medium">0</span>
                                    </span>
                                    <button id="buy-protection-btn" class="bg-primary hover:bg-primary/90 text-white px-4 py-2 rounded-full text-sm font-medium transition-colors disabled:opacity-50 disabled:cursor-not-allowed">
                                        购买
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    <!-- 飞镖卡商品 -->
                    <div class="border border-gray-200 rounded-xl p-5 hover:shadow-md transition-shadow">
                        <div class="flex items-start">
                            <div class="w-12 h-12 bg-accent/20 text-accent rounded-lg flex items-center justify-center flex-shrink-0">
                                <i class="fa fa-crosshairs text-2xl"></i>
                            </div>
                            <div class="ml-4 flex-grow">
                                <div class="flex justify-between items-start">
                                    <h4 class="font-bold text-lg">飞镖卡</h4>
                                    <span class="bg-coin/10 text-coin px-2 py-1 rounded text-sm font-medium">
                                        <i class="fa fa-diamond mr-1"></i>30金币
                                    </span>
                                </div>
                                <p class="text-gray-600 text-sm mt-2">
                                    启用后永久生效，每次跳转阶段时，跳转后的阶段会有一个随机任务（非十号任务）
                                    被标记为已完成状态，并标注为"被飞镖卡破坏的任务"。
                                </p>
                                <div class="mt-3 flex justify-between items-center">
                                    <span class="text-sm text-gray-500">
                                        状态: <span id="shop-dart-status" class="font-medium">未启用</span>
                                    </span>
                                    <button id="buy-dart-btn" class="bg-primary hover:bg-primary/90 text-white px-4 py-2 rounded-full text-sm font-medium transition-colors disabled:opacity-50 disabled:cursor-not-allowed">
                                        购买并启用
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    <!-- 新增：喵喵卡 -->
                    <div class="border border-gray-200 rounded-xl p-5 hover:shadow-md transition-shadow">
                        <div class="flex items-start">
                            <div class="w-12 h-12 bg-cat/20 text-cat rounded-lg flex items-center justify-center flex-shrink-0">
                                <i class="fa fa-paw text-2xl"></i>
                            </div>
                            <div class="ml-4 flex-grow">
                                <div class="flex justify-between items-start">
                                    <h4 class="font-bold text-lg">喵喵卡</h4>
                                    <span class="bg-coin/10 text-coin px-2 py-1 rounded text-sm font-medium">
                                        <i class="fa fa-diamond mr-1"></i>40金币
                                    </span>
                                </div>
                                <p class="text-gray-600 text-sm mt-2">
                                    启用后永久生效，在猫猫挑战中减少5秒挑战时间。无论是温柔猫猫、坏蛋猫猫还是狡猾猫猫，
                                    挑战时间都会减少5秒。
                                </p>
                                <div class="mt-3 flex justify-between items-center">
                                    <span class="text-sm text-gray-500">
                                        状态: <span id="shop-meow-status" class="font-medium">未启用</span>
                                    </span>
                                    <button id="buy-meow-btn" class="bg-primary hover:bg-primary/90 text-white px-4 py-2 rounded-full text-sm font-medium transition-colors disabled:opacity-50 disabled:cursor-not-allowed">
                                        购买并启用
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    <!-- 新增：连弩卡 -->
                    <div class="border border-gray-200 rounded-xl p-5 hover:shadow-md transition-shadow">
                        <div class="flex items-start">
                            <div class="w-12 h-12 bg-success/20 text-success rounded-lg flex items-center justify-center flex-shrink-0">
                                <i class="fa fa-bolt text-2xl"></i>
                            </div>
                            <div class="ml-4 flex-grow">
                                <div class="flex justify-between items-start">
                                    <h4 class="font-bold text-lg">连弩卡</h4>
                                    <span class="bg-coin/10 text-coin px-2 py-1 rounded text-sm font-medium">
                                        <i class="fa fa-diamond mr-1"></i>50金币
                                    </span>
                                </div>
                                <p class="text-gray-600 text-sm mt-2">
                                    启用后永久生效，每当玩家完成任务时，有50%的几率额外完成一个当前阶段未完成的任务
                                    （非十号任务）。此效果不会触发猫猫挑战。
                                </p>
                                <div class="mt-3 flex justify-between items-center">
                                    <span class="text-sm text-gray-500">
                                        状态: <span id="shop-crossbow-status" class="font-medium">未启用</span>
                                    </span>
                                    <button id="buy-crossbow-btn" class="bg-primary hover:bg-primary/90 text-white px-4 py-2 rounded-full text-sm font-medium transition-colors disabled:opacity-50 disabled:cursor-not-allowed">
                                        购买并启用
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- 编辑奖池选项模态框 -->
    <div id="edit-pool-modal" class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center hidden">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-4xl max-h-[90vh] overflow-y-auto">
            <div class="p-6 border-b">
                <div class="flex justify-between items-center">
                    <h3 class="text-xl font-bold text-secondary">编辑任务选项</h3>
                    <button id="close-pool-modal" class="text-gray-500 hover:text-gray-700">
                        <i class="fa fa-times text-xl"></i>
                    </button>
                </div>
                <p class="text-gray-500 mt-1">当前编辑：<span id="editing-pool-text">阶段 1</span></p>
            </div>
            
            <div class="p-6">
                <div id="edit-options-container" class="space-y-4">
                    <!-- 编辑选项将通过JS动态生成 -->
                </div>
            </div>
            
            <div class="p-6 border-t flex justify-end">
                <button id="save-pool-options" class="bg-primary hover:bg-primary/90 text-white px-6 py-2 rounded-full font-medium transition-colors">
                    保存更改
                </button>
            </div>
        </div>
    </div>

    <!-- 编辑星桶任务模态框 -->
    <div id="edit-star-modal" class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center hidden">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-2xl max-h-[90vh] overflow-y-auto">
            <div class="p-6 border-b">
                <div class="flex justify-between items-center">
                    <h3 class="text-xl font-bold text-secondary">编辑额外挑战</h3>
                    <button id="close-star-modal" class="text-gray-500 hover:text-gray-700">
                        <i class="fa fa-times text-xl"></i>
                    </button>
                </div>
            </div>
            
            <div class="p-6">
                <div id="star-tasks-container" class="space-y-3">
                    <!-- 星桶任务将通过JS动态生成 -->
                </div>
                
                <div class="mt-4">
                    <button id="add-star-task" class="bg-secondary hover:bg-secondary/90 text-white px-4 py-2 rounded-full text-sm font-medium transition-colors">
                        <i class="fa fa-plus mr-1"></i> 添加额外挑战
                    </button>
                </div>
            </div>
            
            <div class="p-6 border-t flex justify-end">
                <button id="save-star-tasks" class="bg-primary hover:bg-primary/90 text-white px-6 py-2 rounded-full font-medium transition-colors">
                    保存更改
                </button>
            </div>
        </div>
    </div>

    <!-- 帮助模态框 -->
    <div id="help-modal" class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center hidden">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-2xl max-h-[90vh] overflow-y-auto">
            <div class="p-6 border-b">
                <div class="flex justify-between items-center">
                    <h3 class="text-xl font-bold text-secondary">使用帮助</h3>
                    <button id="close-help-modal" class="text-gray-500 hover:text-gray-700">
                        <i class="fa fa-times text-xl"></i>
                    </button>
                </div>
            </div>
            
            <div class="p-6 space-y-4">
                <div>
                    <h4 class="font-bold text-lg text-primary mb-2">基本操作</h4>
                    <p>1. 这是一个由AI编写的健身游戏，通过完成任务，达到最后的阶段通关！  </p>
                    <p>2. 这个游戏没有固定的玩法需要由玩家自行在游戏前设定一个规则，这里以举哑铃为例，开始在游戏中抽取任务每一组的意思是完成五十个上下举重或者做到力竭算作完成一组，休息片刻后进行下一组直到完成任务所描述的组数。之后点击标记完成按钮继续抽取下一个任务。直到抽到最后一个阶段的最后一个任务即可通关，通关之后请随意发挥。 </p>
                    <p>3. 任务描述中有一些附加的描述请各位自行理解，比如反手的意思是用非惯用手举哑铃，反握的意思是反握哑铃。同时还有附加挑战 默认玩法是在他所规定的姿势下完成任务。  </p>
                    <p>4. 在默认玩法上仅可以通过抽取任务来获得任务，直接选择阶段和选择任务算作是作弊哦。</p>
                    <p>5. 如果已经彻底力竭则视为挑战失败，欢迎充分休息后再来挑战！ </p>
                    <p>6. 希望大家都能享受运动的乐趣，早日练出麒麟臂，但一定要量力而行不要勉强自己◟[˳_˳]ʌ˽ʌ </p>
                    <p>7. 每完成1组任务，可获得1个金币，同时增加一点污染指数 ，污染指数越大怪物就会越强。</p>
                    <p>8. 作者 deepseek 豆包     B站：繁星所向_众望所归 </p>
                </div>
                
                <div>
                    <h4 class="font-bold text-lg text-primary mb-2">猫猫挑战</h4>
                    <p>1. 温柔猫猫：20秒，45BPM，5金币奖励</p>
                    <p>2. 坏蛋猫猫：15秒，90BPM，10金币奖励</p>
                    <p>3. 狡猾猫猫：20秒，45BPM（每4秒增加15BPM），8金币奖励</p>
                    <p>4. 挑战成功可降低10%污染指数并获得金币</p>
                    <p>5. 逃避会提高10%污染指数，任务仍需手动完成</p>
                </div>
                
                <div>
                    <h4 class="font-bold text-lg text-primary mb-2">道具商店</h4>
                    <p>1. 点击顶部导航栏的商店图标进入道具商店</p>
                    <p>2. 保护卡（10金币）：自动使用，可免疫一次跳转至之前阶段的效果</p>
                    <p>3. 飞镖卡（30金币）：永久生效，每次跳转阶段后自动完成一个随机非十号任务</p>
                    <p>4. 喵喵卡（40金币）：永久生效，在猫猫挑战中减少5秒时间</p>
                    <p>5. 连弩卡（50金币）：永久生效，完成任务时有50%几率额外完成一个当前阶段未完成的任务（非十号任务）</p>
                    <p>6. 通过十号任务跳转阶段时，所有保护卡将会清零</p>
                </div>
                
                <div>
                    <h4 class="font-bold text-lg text-primary mb-2">自定义设置</h4>
                    <p>1. 点击"编辑任务"可以修改当前阶段的任务内容</p>
                    <p>2. 每个任务可以设置是否"进入下一阶段"或"跳转到其他阶段"</p>
                    <p>3. 点击"编辑"可以修改额外挑战内容</p>
                </div>
                
                <div>
                    <h4 class="font-bold text-lg text-primary mb-2">阶段导航</h4>
                    <p>1. 可以通过底部的阶段导航切换不同的阶段</p>
                    <p>2. 某些任务完成后会自动进入下一阶段或跳转到指定阶段</p>
                    <p>3. 进入下一阶段后，所有阶段的任务完成状态将被重置</p>
                </div>
            </div>
        </div>
    </div>

    <!-- 通关成功模态框 -->
    <div id="game-clear-modal" class="fixed inset-0 bg-black/70 z-50 flex items-center justify-center hidden">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-md p-8 text-center">
            <div class="mb-6">
                <div class="inline-block bg-primary/10 text-primary p-4 rounded-full">
                    <i class="fa fa-trophy text-5xl"></i>
                </div>
                <h3 class="text-2xl font-bold mt-4">恭喜通关！</h3>
                <p class="text-gray-600 mt-2">你成功完成了所有阶段的挑战</p>
            </div>
            
            <div class="bg-gray-50 rounded-xl p-6 mb-6 text-left space-y-3">
                <div class="flex justify-between">
                    <span class="text-gray-600">完成总任务数</span>
                    <span class="font-bold" id="final-tasks-count">0</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-gray-600">完成总组数</span>
                    <span class="font-bold" id="final-groups-count">0</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-gray-600">获得总金币</span>
                    <span class="font-bold text-coin" id="final-coins-count">0</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-gray-600">最终污染指数</span>
                    <span class="font-bold text-purple" id="final-pollution-index">0%</span>
                </div>
            </div>
            
            <button id="restart-game-btn" class="w-full bg-primary hover:bg-primary/90 text-white py-3 rounded-xl font-medium transition-colors">
                重新开始游戏
            </button>
        </div>
    </div>

    <footer class="bg-dark text-white py-6">
        <div class="container mx-auto px-4 text-center">
            <p>运动任务抽奖发布器 &copy; 2023</p>
            <p class="text-gray-400 text-sm mt-1">设计与开发</p>
        </div>
    </footer>

    <script>
        // 生成运动任务文本 - 根据新阶段规则调整组数生成
        function generateExerciseText(poolNumber, index) {
            let groupCount;
            
            // 根据不同阶段应用不同的组数规则
            if (poolNumber === 1) {
                // 第一层（阶段1）：最多2组，只有3个2组任务，其余都是1组
                // 确保索引1、3、5为2组，其余为1组
                if ([1, 3, 5].includes(index)) {
                    groupCount = 2;
                } else {
                    groupCount = 1;
                }
            } else if (poolNumber === 2) {
                // 第二层（阶段2）：最多3组，只有1个3组任务
                // 确保索引7为3组，其余为1或2组
                if (index === 7) {
                    groupCount = 3;
                } else {
                    // 其余任务随机1或2组，偏向1组
                    groupCount = Math.random() < 0.7 ? 1 : 2;
                }
            } else if (poolNumber === 3) {
                // 第三层（阶段3）：最多3组
                if (index === 0) groupCount = 1;
                else if (index === 1) groupCount = 2;
                else if (index === 2) groupCount = 3;
                else {
                    // 随机1-3组，概率均衡
                    const random = Math.random();
                    if (random < 0.34) groupCount = 1;
                    else if (random < 0.67) groupCount = 2;
                    else groupCount = 3;
                }
            } else {
                // 其他阶段按原规则
                const random = Math.random();
                
                // 确保每个奖池都有1-5至少一个
                if (index === 0) groupCount = 1;
                else if (index === 1) groupCount = 2;
                else if (index === 2) groupCount = 3;
                else if (index === 3) groupCount = 4;
                else if (index === 4) groupCount = 5;
                else {
                    // 根据奖池调整概率分布
                    switch(poolNumber) {
                        case 4: // 阶段4：高组数概率高
                            if (random < 0.05) groupCount = 1;
                            else if (random < 0.2) groupCount = 2;
                            else if (random < 0.4) groupCount = 3;
                            else if (random < 0.7) groupCount = 4;
                            else groupCount = 5;
                            break;
                        case 5: // 阶段5：最高概率
                            if (random < 0.05) groupCount = 1;
                            else if (random < 0.15) groupCount = 2;
                            else if (random < 0.3) groupCount = 3;
                            else if (random < 0.6) groupCount = 4;
                            else groupCount = 5;
                            break;
                    }
                }
            }
            
            // 基础文本
            let text = `做${groupCount}组`;
            
            // 30%几率添加反手
            const hasBackhand = Math.random() < 0.3;
            // 20%几率添加反握
            const hasReverseGrip = Math.random() < 0.2;
            
            if (hasBackhand) text += "，反手";
            if (hasReverseGrip) text += "，反握";
            
            // 第五阶段的十号任务添加特殊标识
            if (poolNumber === 5 && index === 9) {
                text += "（最终挑战）";
            }
            
            return text;
        }
        
        // 生成奖池数据
        function generatePools() {
            const pools = [];
            
            for (let poolNumber = 1; poolNumber <= 5; poolNumber++) {
                const pool = [];
                
                // 计算需要多少个跳转至之前奖池的任务
                const jumpCount = poolNumber === 1 ? 0 : 
                                 poolNumber === 2 ? 1 :
                                 poolNumber === 3 ? 2 :
                                 poolNumber === 4 ? 3 :
                                 poolNumber === 5 ? 4 : 0;
                
                // 生成10个任务
                for (let i = 0; i < 10; i++) {
                    // 第10个任务(索引9)固定为进入下一阶段
                    const isLast = i === 9;
                    let nextStage = isLast && poolNumber < 5;
                    let jumpToPool = null;
                    
                    // 为非最后一个任务随机分配跳转属性
                    if (!isLast && jumpCount > 0 && poolNumber > 1) {
                        // 前面的jumpCount个任务会随机跳转到之前的奖池
                        if (i < jumpCount) {
                            // 随机选择一个之前的奖池
                            jumpToPool = Math.floor(Math.random() * (poolNumber - 1)) + 1;
                            nextStage = false; // 有跳转就不能进入下一阶段
                        }
                    }
                    
                    pool.push({
                        id: `pool${poolNumber}-option${i+1}`,
                        text: generateExerciseText(poolNumber, i),
                        nextStage: nextStage,
                        jumpToPool: jumpToPool,
                        completed: false,
                        completedByDart: false, // 标记是否被飞镖卡完成
                        groupCount: parseInt(generateExerciseText(poolNumber, i).match(/\d+/)[0]), // 存储组数
                        catType: null // 猫的类型，null表示无猫，'gentle'表示温柔猫猫，'bad'表示坏蛋猫猫，'cunning'表示狡猾猫猫
                    });
                }
                
                pools.push(pool);
            }
            
            return pools;
        }

        // 应用状态管理
        const appState = {
            currentPool: 1,
            pools: generatePools(), // 使用生成的运动任务
            // 额外挑战默认设置为运动主题
            starTasks: [
                "站着进行",
                "跪着进行",
                "M开腿蹲着进行",
                "M开腿坐着进行",
                "头比屁股低的姿势进行",
                "与深蹲同时进行",
                "坐着进行",
                "坐着进行",
                "坐着进行",
                "坐着进行"
            ],
            lastDrawnOption: null,
            lastStarTask: null,
            taskInProgress: false,
            gameCompleted: false, // 游戏是否已通关
            // 计数器
            completedTasksCount: 0,
            completedGroupsCount: 0,
            // 经济系统
            coins: 0, // 金币数量
            // 道具系统
            protectionCards: 0,
            protectionCardPrice: 10, // 保护卡价格为10金币
            dartCardEnabled: false, // 飞镖卡是否启用
            dartCardPrice: 30, // 飞镖卡价格为30金币
            // 新增道具
            meowCardEnabled: false, // 喵喵卡是否启用
            meowCardPrice: 40, // 喵喵卡价格为40金币
            crossbowCardEnabled: false, // 连弩卡是否启用
            crossbowCardPrice: 50, // 连弩卡价格为50金币
            // 污染指数
            pollutionIndex: 20
        };

        // DOM 元素引用
        const elements = {
            currentPoolEl: document.getElementById('current-pool'),
            prizeOptionsEl: document.getElementById('prize-options'),
            drawResultEl: document.getElementById('draw-result'),
            completeTaskBtn: document.getElementById('complete-task-btn'),
            attackMonsterBtn: document.getElementById('attack-monster-btn'),
            escapeMonsterBtn: document.getElementById('escape-monster-btn'),
            poolStatusIndicator: document.getElementById('pool-status-indicator'),
            poolStatusText: document.getElementById('pool-status-text'),
            starTaskResultEl: document.getElementById('star-task-result'),
            starTasksCountEl: document.getElementById('star-tasks-count'),
            drawRandomBtn: document.getElementById('draw-random-btn'),
            poolNavBtns: document.querySelectorAll('.pool-nav-btn'),
            editPoolBtn: document.getElementById('edit-pool-btn'),
            editPoolModal: document.getElementById('edit-pool-modal'),
            closePoolModal: document.getElementById('close-pool-modal'),
            editOptionsContainer: document.getElementById('edit-options-container'),
            savePoolOptions: document.getElementById('save-pool-options'),
            editingPoolText: document.getElementById('editing-pool-text'),
            helpBtn: document.getElementById('help-btn'),
            helpModal: document.getElementById('help-modal'),
            closeHelpModal: document.getElementById('close-help-modal'),
            mainHeader: document.getElementById('main-header'),
            // 计数器元素
            completedTasksCountEl: document.getElementById('completed-tasks-count'),
            completedGroupsCountEl: document.getElementById('completed-groups-count'),
            mobileCompletedTasksCountEl: document.getElementById('mobile-completed-tasks-count'),
            mobileCompletedGroupsCountEl: document.getElementById('mobile-completed-groups-count'),
            // 金币元素
            coinsCountEl: document.getElementById('coins-count'),
            mobileCoinsCountEl: document.getElementById('mobile-coins-count'),
            // 保护卡元素
            protectionCardsCountEl: document.getElementById('protection-cards-count'),
            mobileProtectionCardsCountEl: document.getElementById('mobile-protection-cards-count'),
            // 飞镖卡元素
            dartCardStatusEl: document.getElementById('dart-card-status'),
            mobileDartCardStatusEl: document.getElementById('mobile-dart-card-status'),
            // 商店元素
            shopBtn: document.getElementById('shop-btn'),
            shopModal: document.getElementById('shop-modal'),
            closeShopModal: document.getElementById('close-shop-modal'),
            shopCoinsCountEl: document.getElementById('shop-coins-count'),
            shopProtectionCountEl: document.getElementById('shop-protection-count'),
            shopDartStatusEl: document.getElementById('shop-dart-status'),
            shopMeowStatusEl: document.getElementById('shop-meow-status'),
            shopCrossbowStatusEl: document.getElementById('shop-crossbow-status'),
            buyProtectionBtn: document.getElementById('buy-protection-btn'),
            buyDartBtn: document.getElementById('buy-dart-btn'),
            buyMeowBtn: document.getElementById('buy-meow-btn'),
            buyCrossbowBtn: document.getElementById('buy-crossbow-btn'),
            // 污染指数相关元素
            pollutionIndexEl: document.getElementById('pollution-index'),
            starIconEl: document.getElementById('star-icon'),
            // 节拍器挑战相关元素
            beatChallengeModal: document.getElementById('beat-challenge-modal'),
            closeChallengeModal: document.getElementById('close-challenge-modal'),
            beatIndicator: document.getElementById('beat-indicator'),
            challengeTimer: document.getElementById('challenge-timer'),
            challengeResult: document.getElementById('challenge-result'),
            challengeTotalTime: document.getElementById('challenge-total-time'),
            timeIncreaseNotice: document.getElementById('time-increase-notice'),
            bpmIncreaseNotice: document.getElementById('bpm-increase-notice'),
            meowCardEffect: document.getElementById('meow-card-effect'),
            challengeTitle: document.getElementById('challenge-title'),
            challengeBpm: document.getElementById('challenge-bpm'),
            currentBpm: document.getElementById('current-bpm'),
            // 通关相关元素
            gameClearModal: document.getElementById('game-clear-modal'),
            finalTasksCountEl: document.getElementById('final-tasks-count'),
            finalGroupsCountEl: document.getElementById('final-groups-count'),
            finalCoinsCountEl: document.getElementById('final-coins-count'),
            finalPollutionIndexEl: document.getElementById('final-pollution-index'),
            restartGameBtn: document.getElementById('restart-game-btn')
        };

        // 更新计数器显示
        function updateCounters() {
            // 更新任务和组数计数器
            elements.completedTasksCountEl.textContent = appState.completedTasksCount;
            elements.completedGroupsCountEl.textContent = appState.completedGroupsCount;
            elements.mobileCompletedTasksCountEl.textContent = appState.completedTasksCount;
            elements.mobileCompletedGroupsCountEl.textContent = appState.completedGroupsCount;
            
            // 更新金币计数器
            elements.coinsCountEl.textContent = appState.coins;
            elements.mobileCoinsCountEl.textContent = appState.coins;
            
            // 更新保护卡计数器
            elements.protectionCardsCountEl.textContent = appState.protectionCards;
            elements.mobileProtectionCardsCountEl.textContent = appState.protectionCards;
            
            // 更新飞镖卡状态
            const dartStatus = appState.dartCardEnabled ? "已启用" : "未启用";
            elements.dartCardStatusEl.textContent = dartStatus;
            elements.mobileDartCardStatusEl.textContent = dartStatus;
            
            // 更新商店中的显示
            elements.shopCoinsCountEl.textContent = appState.coins;
            elements.shopProtectionCountEl.textContent = appState.protectionCards;
            elements.shopDartStatusEl.textContent = dartStatus;
            elements.shopMeowStatusEl.textContent = appState.meowCardEnabled ? "已启用" : "未启用";
            elements.shopCrossbowStatusEl.textContent = appState.crossbowCardEnabled ? "已启用" : "未启用";
            
            // 更新购买按钮状态
            elements.buyProtectionBtn.disabled = appState.coins < appState.protectionCardPrice || appState.gameCompleted;
            elements.buyDartBtn.disabled = appState.coins < appState.dartCardPrice || appState.dartCardEnabled || appState.gameCompleted;
            elements.buyMeowBtn.disabled = appState.coins < appState.meowCardPrice || appState.meowCardEnabled || appState.gameCompleted;
            elements.buyCrossbowBtn.disabled = appState.coins < appState.crossbowCardPrice || appState.crossbowCardEnabled || appState.gameCompleted;
            
            // 更新污染指数显示
            updatePollutionIndex();
        }
        
        // 更新污染指数显示并改变星星颜色
        function updatePollutionIndex() {
            elements.pollutionIndexEl.textContent = `${appState.pollutionIndex}%`;
            
            // 根据污染指数改变星星颜色
            // 30%开始逐渐从黄色变为紫色
            let starColor;
            if (appState.pollutionIndex < 30) {
                // 30%以下保持黄色
                starColor = '#FFC107';
            } else {
                // 30%及以上开始混合黄色和紫色
                // 计算混合比例 (30% -> 0% 紫色, 100% -> 100% 紫色)
                const ratio = Math.min(1, (appState.pollutionIndex - 30) / 70);
                
                // 黄色 (#FFC107) RGB: 255, 193, 7
                // 紫色 (#9C27B0) RGB: 156, 39, 176
                const r = Math.round(255 - (255 - 156) * ratio);
                const g = Math.round(193 - (193 - 39) * ratio);
                const b = Math.round(7 + (176 - 7) * ratio);
                
                starColor = `rgb(${r}, ${g}, ${b})`;
            }
            
            elements.starIconEl.style.color = starColor;
        }

        // 初始化应用
        function initApp() {
            renderPrizeOptions();
            updateStarTasksCount();
            updateCounters(); // 初始化计数器显示
            setupEventListeners();
        }

        // 渲染奖池选项
        function renderPrizeOptions() {
            const currentOptions = appState.pools[appState.currentPool - 1];
            elements.prizeOptionsEl.innerHTML = '';
            
            currentOptions.forEach((option, index) => {
                const optionEl = document.createElement('div');
                // 根据猫的类型设置不同样式
                let catClass = 'border-gray-200';
                if (option.catType === 'gentle') {
                    catClass = 'border-cat';
                } else if (option.catType === 'bad') {
                    catClass = 'border-badcat';
                } else if (option.catType === 'cunning') {
                    catClass = 'border-cunningcat';
                }
                
                // 游戏通关后所有选项不可点击
                const gameCompletedClass = appState.gameCompleted ? 'opacity-50 cursor-not-allowed' : '';
                
                optionEl.className = `bg-white ${catClass} rounded-xl p-3 prize-option-hover cursor-pointer ${option.completed ? 'opacity-50' : ''} ${gameCompletedClass}`;
                
                // 确定完成标记文本
                let completedText = '';
                if (option.completed) {
                    if (option.catType && !option.completedByDart) {
                        let catName = '';
                        if (option.catType === 'gentle') catName = '温柔猫猫';
                        else if (option.catType === 'bad') catName = '坏蛋猫猫';
                        else if (option.catType === 'cunning') catName = '狡猾猫猫';
                        
                        completedText = `<div class="mt-2 text-success text-xs"><i class="fa fa-check-circle mr-1"></i> ${catName}已被安抚</div>`;
                    } else if (option.completedByDart) {
                        completedText = '<div class="mt-2 text-accent text-xs"><i class="fa fa-crosshairs mr-1"></i> 被飞镖卡破坏</div>';
                    } else {
                        completedText = '<div class="mt-2 text-success text-xs"><i class="fa fa-check-circle mr-1"></i> 已完成</div>';
                    }
                }
                
                // 猫的标记
                let catIndicator = '';
                if (option.catType === 'gentle') {
                    catIndicator = '<div class="flex items-center justify-center mt-1"><span class="bg-cat/10 text-cat px-1.5 py-0.5 rounded text-xs"><i class="fa fa-paw mr-0.5"></i>温柔猫猫</span></div>';
                } else if (option.catType === 'bad') {
                    catIndicator = '<div class="flex items-center justify-center mt-1"><span class="bg-badcat/10 text-badcat px-1.5 py-0.5 rounded text-xs"><i class="fa fa-paw mr-0.5"></i>坏蛋猫猫</span></div>';
                } else if (option.catType === 'cunning') {
                    catIndicator = '<div class="flex items-center justify-center mt-1"><span class="bg-cunningcat/10 text-cunningcat px-1.5 py-0.5 rounded text-xs"><i class="fa fa-paw mr-0.5"></i>狡猾猫猫</span></div>';
                }
                
                // 特殊任务标记
                let specialTaskBadge = '';
                if (appState.currentPool === 5 && index === 9) {
                    specialTaskBadge = '<div class="mt-1 inline-block bg-primary/10 text-primary px-1.5 py-0.5 rounded text-xs">最终挑战</div>';
                }
                
                optionEl.innerHTML = `
                    <div class="text-center">
                        <div class="w-8 h-8 bg-primary/10 text-primary rounded-full flex items-center justify-center font-bold mx-auto mb-2">
                            ${index + 1}
                        </div>
                        <p class="text-sm mb-1 line-clamp-2">${option.text}</p>
                        <div class="flex items-center justify-center space-x-1 text-xs text-gray-500 mb-1">
                            ${option.nextStage ? '<span class="bg-success/10 text-success px-1.5 py-0.5 rounded"><i class="fa fa-arrow-right mr-0.5"></i>下一阶段</span>' : ''}
                            ${option.jumpToPool ? `<span class="bg-accent/10 text-accent px-1.5 py-0.5 rounded"><i class="fa fa-exchange mr-0.5"></i>阶段${option.jumpToPool}</span>` : ''}
                        </div>
                        ${specialTaskBadge}
                        ${catIndicator}
                        ${completedText}
                    </div>
                `;
                
                optionEl.addEventListener('click', () => {
                    if (!appState.taskInProgress && !option.completed && !appState.gameCompleted) {
                        drawOption(index);
                    }
                });
                
                elements.prizeOptionsEl.appendChild(optionEl);
            });
        }

        // 抽取选项
        function drawOption(optionIndex) {
            const currentOptions = appState.pools[appState.currentPool - 1];
            const selectedOption = currentOptions[optionIndex];
            
            // 更新应用状态
            appState.lastDrawnOption = {
                pool: appState.currentPool,
                index: optionIndex,
                ...selectedOption
            };
            appState.taskInProgress = true;
            
            // 抽取附加任务
            drawStarTask();
            
            // 更新UI
            renderDrawResult();
            updatePoolStatus();
            
            // 根据是否有猫显示不同按钮
            if (selectedOption.catType) {
                elements.completeTaskBtn.classList.add('hidden');
                elements.attackMonsterBtn.classList.remove('hidden');
                elements.escapeMonsterBtn.classList.remove('hidden');
                
                // 设置挑战按钮样式和文本
                if (selectedOption.catType === 'gentle') {
                    elements.attackMonsterBtn.className = 'bg-cat hover:bg-cat/90 text-white px-6 py-3 rounded-full font-medium shadow-md hover:shadow-lg transition-all';
                    elements.attackMonsterBtn.innerHTML = '<i class="fa fa-paw mr-2"></i>挑战温柔猫猫';
                } else if (selectedOption.catType === 'bad') {
                    elements.attackMonsterBtn.className = 'bg-badcat hover:bg-badcat/90 text-white px-6 py-3 rounded-full font-medium shadow-md hover:shadow-lg transition-all';
                    elements.attackMonsterBtn.innerHTML = '<i class="fa fa-paw mr-2"></i>挑战坏蛋猫猫';
                } else if (selectedOption.catType === 'cunning') {
                    elements.attackMonsterBtn.className = 'bg-cunningcat hover:bg-cunningcat/90 text-white px-6 py-3 rounded-full font-medium shadow-md hover:shadow-lg transition-all';
                    elements.attackMonsterBtn.innerHTML = '<i class="fa fa-paw mr-2"></i>挑战狡猾猫猫';
                }
            } else {
                elements.completeTaskBtn.classList.remove('hidden');
                elements.attackMonsterBtn.classList.add('hidden');
                elements.escapeMonsterBtn.classList.add('hidden');
                elements.completeTaskBtn.disabled = false;
            }
            
            elements.drawRandomBtn.disabled = true;
        }

        // 随机抽取一个选项
        function drawRandomOption() {
            const currentOptions = appState.pools[appState.currentPool - 1];
            const availableOptions = currentOptions.filter(option => !option.completed);
            
            if (availableOptions.length === 0) {
                alert('当前阶段所有任务已完成！');
                return;
            }
            
            // 添加旋转动画效果
            elements.prizeOptionsEl.classList.add('animate-pulse');
            
            // 延迟后显示结果，模拟随机过程
            setTimeout(() => {
                elements.prizeOptionsEl.classList.remove('animate-pulse');
                const randomIndex = Math.floor(Math.random() * availableOptions.length);
                const originalIndex = currentOptions.findIndex(
                    opt => opt.id === availableOptions[randomIndex].id
                );
                drawOption(originalIndex);
            }, 1500);
        }

        // 抽取星桶任务 - 增加15%概率添加"上"，10%概率添加"下"
        function drawStarTask() {
            if (appState.starTasks.length === 0) {
                appState.lastStarTask = "没有可用的额外挑战";
                return;
            }
            
            const randomIndex = Math.floor(Math.random() * appState.starTasks.length);
            let taskText = appState.starTasks[randomIndex];
            
            // 15%概率添加"上"
            const addUp = Math.random() < 0.15;
            // 10%概率添加"下"
            const addDown = Math.random() < 0.10;
            
            if (addUp) taskText += "，上";
            if (addDown) taskText += "，下";
            
            appState.lastStarTask = taskText;
        }

        // 渲染抽取结果
        function renderDrawResult() {
            if (!appState.lastDrawnOption) return;
            
            const { text, nextStage, jumpToPool, catType } = appState.lastDrawnOption;
            
            // 猫的标记
            let catBadge = '';
            if (catType === 'gentle') {
                catBadge = '<div class="bg-cat/10 text-cat rounded-lg py-1.5 mb-3 inline-block"><i class="fa fa-paw mr-1"></i>发现温柔猫猫！</div>';
            } else if (catType === 'bad') {
                catBadge = '<div class="bg-badcat/10 text-badcat rounded-lg py-1.5 mb-3 inline-block"><i class="fa fa-paw mr-1"></i>发现坏蛋猫猫！</div>';
            } else if (catType === 'cunning') {
                catBadge = '<div class="bg-cunningcat/10 text-cunningcat rounded-lg py-1.5 mb-3 inline-block"><i class="fa fa-paw mr-1"></i>发现狡猾猫猫！</div>';
            }
            
            elements.drawResultEl.innerHTML = `
                <div class="w-full">
                    ${catBadge}
                    <div class="bg-primary/10 text-primary rounded-lg py-2 mb-3 inline-block">
                        <span class="font-medium">抽取到的任务</span>
                    </div>
                    <p class="text-lg mb-4">${text}</p>
                    <div class="flex flex-wrap justify-center gap-2 mb-4">
                        ${nextStage ? '<span class="bg-success/10 text-success px-3 py-1 rounded-full text-sm"><i class="fa fa-arrow-right mr-1"></i>完成后进入下一阶段</span>' : ''}
                        ${jumpToPool ? `<span class="bg-accent/10 text-accent px-3 py-1 rounded-full text-sm"><i class="fa fa-exchange mr-1"></i>完成后跳转到阶段${jumpToPool}</span>` : ''}
                        ${jumpToPool && appState.protectionCards > 0 ? `<span class="bg-warning/10 text-warning px-3 py-1 rounded-full text-sm"><i class="fa fa-shield mr-1"></i>将自动使用保护卡免疫跳转</span>` : ''}
                    </div>
                </div>
            `;
            
            // 显示附加任务
            elements.starTaskResultEl.innerHTML = `
                <div class="bg-warning/10 text-warning rounded-lg py-1.5 mb-2 inline-block text-sm">
                    <i class="fa fa-star mr-1"></i>额外挑战
                </div>
                <p>${appState.lastStarTask}</p>
            `;
        }

        // 更新奖池状态
        function updatePoolStatus() {
            if (appState.taskInProgress) {
                elements.poolStatusIndicator.className = 'w-3 h-3 rounded-full bg-warning ml-3';
                elements.poolStatusText.className = 'ml-2 text-sm text-warning font-medium';
                elements.poolStatusText.textContent = '任务进行中';
            } else {
                elements.poolStatusIndicator.className = 'w-3 h-3 rounded-full bg-success ml-3';
                elements.poolStatusText.className = 'ml-2 text-sm text-success font-medium';
                elements.poolStatusText.textContent = '可抽取';
            }
        }

        // 重置除当前阶段外的所有奖池的完成状态和猫属性
        function resetPoolsExceptCurrent(currentPool) {
            appState.pools.forEach((pool, index) => {
                const poolNumber = index + 1;
                pool.forEach(option => {
                    // 保留飞镖卡完成的任务状态
                    if (!option.completedByDart) {
                        option.completed = false;
                    }
                    
                    // 只清除非当前阶段的猫属性
                    if (poolNumber !== currentPool) {
                        option.catType = null;
                    }
                });
            });
        }

        // 飞镖卡效果：随机完成一个非十号任务
        function applyDartCardEffect(poolNumber) {
            if (!appState.dartCardEnabled) return;
            
            const targetPool = appState.pools[poolNumber - 1];
            // 排除十号任务（索引9）
            const eligibleOptions = targetPool.filter((_, index) => index !== 9 && !_.completed);
            
            if (eligibleOptions.length > 0) {
                const randomIndex = Math.floor(Math.random() * eligibleOptions.length);
                const optionToComplete = eligibleOptions[randomIndex];
                const originalIndex = targetPool.findIndex(opt => opt.id === optionToComplete.id);
                
                // 标记为被飞镖卡完成，同时移除猫属性
                targetPool[originalIndex].completed = true;
                targetPool[originalIndex].completedByDart = true;
                targetPool[originalIndex].catType = null;
            }
        }

        // 阶段跳转时随机添加猫属性，包括狡猾猫猫
        function addRandomCat(poolNumber) {
            // 计算猫生成概率：50% + 当前污染指数百分比
            const baseProbability = 0.5; // 50%基础概率
            const pollutionFactor = appState.pollutionIndex / 100; // 污染指数转换为0-1
            const totalProbability = Math.min(1, baseProbability + pollutionFactor); // 最大100%
            
            // 第一次判定
            if (Math.random() <= totalProbability) {
                const targetPool = appState.pools[poolNumber - 1];
                // 只选择非十号任务（索引9）、未完成、未被飞镖卡完成且没有猫属性的任务
                const eligibleOptions = targetPool.filter((_, index) => 
                    index !== 9 && !_.completed && !_.completedByDart && !_.catType
                );
                
                if (eligibleOptions.length > 0) {
                    // 随机选择一个任务添加猫属性
                    const randomIndex = Math.floor(Math.random() * eligibleOptions.length);
                    const optionToCat = eligibleOptions[randomIndex];
                    const originalIndex = targetPool.findIndex(opt => opt.id === optionToCat.id);
                    
                    let catType = 'gentle'; // 默认是温柔猫猫
                    
                    // 除第一阶段外，有15%概率是狡猾猫猫，替换温柔猫猫
                    if (poolNumber !== 1 && Math.random() <= 0.15) {
                        catType = 'cunning'; // 狡猾猫猫
                    }
                    // 第三、四阶段有30%概率是坏蛋猫猫
                    else if ((poolNumber === 3 || poolNumber === 4) && Math.random() <= 0.3) {
                        catType = 'bad'; // 坏蛋猫猫
                    }
                    
                    targetPool[originalIndex].catType = catType;
                    
                    // 显示相应的消息
                    if (catType === 'gentle') {
                        showMessage("注意！本阶段出现了温柔猫猫");
                    } else if (catType === 'bad') {
                        showMessage("小心！本阶段出现了坏蛋猫猫");
                    } else if (catType === 'cunning') {
                        showMessage("警惕！本阶段出现了狡猾猫猫");
                    }
                    
                    // 继续尝试添加更多猫，传入已判定次数1
                    continueAddingCats(poolNumber, pollutionFactor, 1);
                } else {
                    // 如果没有符合条件的任务，显示提示
                    showMessage("本阶段应该出现猫猫，但没有可用任务");
                }
            }
        }
        
        // 继续添加猫直到判定失败或达到上限
        function continueAddingCats(poolNumber, pollutionFactor, attemptCount) {
            const targetPool = appState.pools[poolNumber - 1];
            // 统计当前已有猫的任务数量
            const catCount = targetPool.filter(opt => opt.catType).length;
            
            // 限制最多8个有猫的任务
            if (catCount >= 8) return;
            
            // 计算当前尝试的概率：污染指数概率 - 当前阶段已判定次数 × 10%
            let currentProbability = pollutionFactor - (attemptCount * 0.1);
            // 确保概率不会小于0
            currentProbability = Math.max(0, currentProbability);
            
            // 再次判定
            if (Math.random() <= currentProbability) {
                // 选择没有猫属性、非十号任务且未完成的任务
                const eligibleOptions = targetPool.filter((_, index) => 
                    index !== 9 && !_.completed && !_.completedByDart && !_.catType
                );
                
                if (eligibleOptions.length > 0) {
                    // 随机选择一个任务添加猫属性
                    const randomIndex = Math.floor(Math.random() * eligibleOptions.length);
                    const optionToCat = eligibleOptions[randomIndex];
                    const originalIndex = targetPool.findIndex(opt => opt.id === optionToCat.id);
                    
                    let catType = 'gentle'; // 默认是温柔猫猫
                    
                    // 除第一阶段外，有15%概率是狡猾猫猫
                    if (poolNumber !== 1 && Math.random() <= 0.15) {
                        catType = 'cunning'; // 狡猾猫猫
                    }
                    // 第三、四阶段有30%概率是坏蛋猫猫
                    else if ((poolNumber === 3 || poolNumber === 4) && Math.random() <= 0.3) {
                        catType = 'bad'; // 坏蛋猫猫
                    }
                    
                    targetPool[originalIndex].catType = catType;
                    
                    // 递归继续尝试添加，增加尝试计数
                    continueAddingCats(poolNumber, pollutionFactor, attemptCount + 1);
                }
            }
        }

        // 连弩卡效果：随机完成一个当前阶段未完成的任务（非十号任务）
        function applyCrossbowCardEffect() {
            if (!appState.crossbowCardEnabled) return;
            
            const currentOptions = appState.pools[appState.currentPool - 1];
            // 排除十号任务（索引9）和已完成的任务
            const eligibleOptions = currentOptions.filter((_, index) => 
                index !== 9 && !_.completed && !_.completedByDart
            );
            
            if (eligibleOptions.length > 0) {
                const randomIndex = Math.floor(Math.random() * eligibleOptions.length);
                const optionToComplete = eligibleOptions[randomIndex];
                const originalIndex = currentOptions.findIndex(opt => opt.id === optionToComplete.id);
                
                // 标记为已完成
                currentOptions[originalIndex].completed = true;
                
                // 更新计数器和金币
                appState.completedTasksCount++;
                appState.completedGroupsCount += optionToComplete.groupCount;
                appState.coins += optionToComplete.groupCount; // 每完成1组获得1金币
                
                // 每完成一组，污染指数上升1%
                appState.pollutionIndex += optionToComplete.groupCount;
                
                showMessage(`连弩卡生效！任务${originalIndex + 1}自动完成`);
                
                // 更新UI
                updateCounters();
                renderPrizeOptions();
            }
        }

        // 标记任务为已完成 - 更新计数器
        function completeCurrentTask() {
            if (!appState.lastDrawnOption) return;
            
            const { pool, index, groupCount, nextStage, jumpToPool } = appState.lastDrawnOption;
            appState.pools[pool - 1][index].completed = true;
            appState.pools[pool - 1][index].completedByDart = false; // 手动完成的任务不标记为飞镖卡完成
            appState.pools[pool - 1][index].catType = null; // 完成后移除猫属性
            
            // 更新计数器和金币
            appState.completedTasksCount++;
            appState.completedGroupsCount += groupCount;
            appState.coins += groupCount; // 每完成1组获得1金币
            
            // 每完成一组，污染指数上升1%
            appState.pollutionIndex += groupCount;
            
            // 检查是否需要处理跳转
            let shouldReset = false;
            let targetPool = pool;
            let jumpPrevented = false;
            
            // 处理进入下一阶段（十号任务）
            if (nextStage && pool < 5) {
                shouldReset = true; // 进入下一阶段时需要重置
                targetPool = pool + 1;
                // 通过十号任务跳转阶段，所有保护卡清零
                const previousProtectionCount = appState.protectionCards;
                appState.protectionCards = 0;
                // 只有当之前保护卡数量大于0时才显示消息
                if (previousProtectionCount > 0) {
                    showMessage("通过十号任务进入新阶段，保护卡已清零");
                }
            }
            // 处理跳转至之前阶段，使用保护卡
            else if (jumpToPool && jumpToPool >= 1 && jumpToPool <= 5) {
                // 如果有保护卡，自动使用
                if (appState.protectionCards > 0) {
                    appState.protectionCards--;
                    jumpPrevented = true;
                    showMessage("已自动使用保护卡，免疫了此次跳转");
                } else {
                    targetPool = jumpToPool;
                    showMessage(`已跳转到阶段${jumpToPool}`);
                }
            }
            
            // 保存重置状态，调整执行顺序
            const needReset = shouldReset;
            
            // 执行跳转
            if (targetPool !== pool && !jumpPrevented) {
                switchToPool(targetPool);
                
                // 如果启用了飞镖卡，应用效果
                applyDartCardEffect(targetPool);
                
                // 阶段跳转时随机添加猫属性
                addRandomCat(targetPool);
            }
            
            // 如果是进入下一阶段，重置除当前阶段外的所有奖池的完成状态
            if (needReset) {
                resetPoolsExceptCurrent(targetPool);
            }
            
            // 检查是否完成了第五阶段的十号任务（最终挑战）
            if (pool === 5 && index === 9) {
                // 标记游戏为已完成
                appState.gameCompleted = true;
                showGameClearScreen();
                return;
            }
            
            // 应用连弩卡效果（50%概率）
            if (appState.crossbowCardEnabled && Math.random() < 0.5) {
                applyCrossbowCardEffect();
            }
            
            // 重置状态
            appState.lastDrawnOption = null;
            appState.lastStarTask = null;
            appState.taskInProgress = false;
            
            // 更新UI
            renderPrizeOptions();
            elements.drawResultEl.innerHTML = '<p class="text-gray-400">点击上方任务选项进行抽取</p>';
            elements.starTaskResultEl.innerHTML = '<p class="text-gray-400">抽取主任务后将获得额外挑战</p>';
            elements.completeTaskBtn.disabled = true;
            elements.attackMonsterBtn.classList.add('hidden');
            elements.escapeMonsterBtn.classList.add('hidden');
            elements.drawRandomBtn.disabled = false;
            updatePoolStatus();
            updateCounters();
            
            // 显示完成动画
            showCompletionAnimation();
        }

        // 挑战猫猫
        function attackMonster() {
            if (!appState.lastDrawnOption) return;
            
            const { catType } = appState.lastDrawnOption;
            let totalTime, baseBpm;
            let timeIncreaseCount = 0; // 时间增加次数，最多4次
            
            // 根据猫的类型设置不同的基础属性
            if (catType === 'gentle') {
                // 温柔猫猫：20秒，45BPM
                totalTime = 20;
                baseBpm = 45;
                
                // 温柔猫猫的时间增加机制
                // 计算时间增加概率（等于当前污染指数百分比）
                const timeIncreaseProbability = appState.pollutionIndex / 100;
                
                // 第一次判定：增加10秒
                if (Math.random() <= timeIncreaseProbability) {
                    totalTime += 10;
                    timeIncreaseCount++;
                    
                    // 后续判定：每次增加1秒，最多4次
                    while (timeIncreaseCount < 4) {
                        if (Math.random() <= timeIncreaseProbability) {
                            totalTime += 1;
                            timeIncreaseCount++;
                        } else {
                            break; // 判定失败，停止增加
                        }
                    }
                }
            } else if (catType === 'bad') {
                // 坏蛋猫猫：15秒，90BPM
                totalTime = 15;
                baseBpm = 90;
                
                // 坏蛋猫猫有BPM增加机制
                let increaseCount = 0; // 增加次数，最多4次
                
                // 计算BPM增加概率（等于当前污染指数百分比）
                const bpmIncreaseProbability = appState.pollutionIndex / 100;
                
                // BPM增加判定逻辑
                while (increaseCount < 4) {
                    if (Math.random() <= bpmIncreaseProbability) {
                        baseBpm += 15;
                        increaseCount++;
                    } else {
                        break; // 判定失败，停止增加
                    }
                }
                
                // 如果有BPM增加，显示提示
                if (increaseCount > 0) {
                    timeIncreaseCount = increaseCount; // 用于显示提示
                }
            } else if (catType === 'cunning') {
                // 狡猾猫猫：20秒，45BPM，每4秒增加15BPM
                totalTime = 20;
                baseBpm = 45;
            }
            
            // 应用喵喵卡效果：减少5秒挑战时间
            if (appState.meowCardEnabled) {
                totalTime = Math.max(5, totalTime - 5); // 确保挑战时间至少为5秒
            }
            
            // 显示节拍器挑战模态框
            elements.beatChallengeModal.classList.remove('hidden');
            elements.challengeResult.classList.add('hidden');
            elements.timeIncreaseNotice.classList.add('hidden');
            elements.bpmIncreaseNotice.classList.add('hidden');
            elements.meowCardEffect.classList.add('hidden');
            
            // 设置挑战标题和样式
            if (catType === 'gentle') {
                elements.challengeTitle.className = 'text-2xl font-bold text-cat flex items-center justify-center';
                elements.challengeTitle.innerHTML = '<i class="fa fa-paw mr-2"></i>温柔猫猫挑战';
                elements.beatIndicator.className = 'beat-box bg-cat';
            } else if (catType === 'bad') {
                elements.challengeTitle.className = 'text-2xl font-bold text-badcat flex items-center justify-center';
                elements.challengeTitle.innerHTML = '<i class="fa fa-paw mr-2"></i>坏蛋猫猫挑战';
                elements.beatIndicator.className = 'beat-box bg-badcat';
            } else if (catType === 'cunning') {
                elements.challengeTitle.className = 'text-2xl font-bold text-cunningcat flex items-center justify-center';
                elements.challengeTitle.innerHTML = '<i class="fa fa-paw mr-2"></i>狡猾猫猫挑战';
                elements.beatIndicator.className = 'beat-box bg-cunningcat';
                // 显示BPM增加提示
                elements.bpmIncreaseNotice.classList.remove('hidden');
            }
            
            // 如果有喵喵卡效果，显示提示
            if (appState.meowCardEnabled) {
                elements.meowCardEffect.classList.remove('hidden');
            }
            
            // 显示总时间和BPM
            elements.challengeTotalTime.textContent = totalTime;
            elements.challengeTimer.textContent = totalTime;
            elements.currentBpm.textContent = baseBpm;
            
            // 如果有增加，显示提示
            if (timeIncreaseCount > 0 && catType !== 'cunning') {
                elements.timeIncreaseNotice.classList.remove('hidden');
                if (catType === 'gentle') {
                    elements.timeIncreaseNotice.innerHTML = `<i class="fa fa-clock-o mr-1"></i>挑战时间增加了${10 + (timeIncreaseCount - 1)}秒！`;
                } else {
                    elements.timeIncreaseNotice.innerHTML = `<i class="fa fa-tachometer mr-1"></i>挑战速度增加了${timeIncreaseCount * 15}BPM！`;
                }
            }
            
            // 重置挑战状态
            elements.closeChallengeModal.disabled = true;
            elements.closeChallengeModal.textContent = "挑战进行中...";
            
            // 创建节拍动画，传入当前BPM
            animateBeat(baseBpm);
            
            // 倒计时器
            let currentBpmValue = baseBpm;
            const timerInterval = setInterval(() => {
                let currentTime = parseInt(elements.challengeTimer.textContent);
                currentTime--;
                elements.challengeTimer.textContent = currentTime;
                
                // 狡猾猫猫每4秒增加15BPM
                if (catType === 'cunning' && currentTime > 0 && currentTime % 4 === 0) {
                    currentBpmValue += 15;
                    elements.currentBpm.textContent = currentBpmValue;
                    // 更新节拍动画速度
                    animateBeat(currentBpmValue);
                }
                
                if (currentTime <= 0) {
                    clearInterval(timerInterval);
                    endBeatChallenge(true, catType);
                }
            }, 1000);
        }

        // 节拍动画 - 根据BPM动态调整速度
        function animateBeat(bpm) {
            const beatBox = elements.beatIndicator;
            let startTime = null;
            const amplitude = 60; // 移动幅度
            const period = 60000 / bpm; // 周期(毫秒)，根据BPM计算
            
            function animate(currentTime) {
                if (!startTime) startTime = currentTime;
                const timeElapsed = currentTime - startTime;
                
                // 计算位置 (正弦曲线)
                const position = Math.sin((timeElapsed / period) * Math.PI * 2) * amplitude;
                
                // 应用位置
                beatBox.style.transform = `translateY(${position}px)`;
                
                // 继续动画，直到挑战结束
                if (elements.beatChallengeModal.classList.contains('hidden') === false && 
                    elements.challengeResult.classList.contains('hidden')) {
                    requestAnimationFrame(animate);
                }
            }
            
            requestAnimationFrame(animate);
        }

        // 结束节拍挑战
        function endBeatChallenge(success, catType) {
            if (success) {
                // 挑战成功
                elements.challengeResult.classList.remove('hidden');
                elements.closeChallengeModal.disabled = false;
                elements.closeChallengeModal.textContent = "完成";
                
                // 更新任务状态
                const { pool, index, groupCount, nextStage, jumpToPool } = appState.lastDrawnOption;
                appState.pools[pool - 1][index].completed = true;
                appState.pools[pool - 1][index].completedByDart = false;
                appState.pools[pool - 1][index].catType = null;
                
                // 更新计数器
                appState.completedTasksCount++;
                appState.completedGroupsCount += groupCount;
                
                // 根据猫的类型给予不同的金币奖励
                let bonusCoins = 0;
                if (catType === 'gentle') bonusCoins = 5;
                else if (catType === 'bad') bonusCoins = 10;
                else if (catType === 'cunning') bonusCoins = 8;
                
                appState.coins += groupCount + bonusCoins; // 基础金币 + 挑战奖励金币
                
                // 污染指数降低10%
                appState.pollutionIndex = Math.max(0, appState.pollutionIndex - 10);
                
                // 处理跳转逻辑
                let shouldReset = false;
                let targetPool = pool;
                let jumpPrevented = false;
                
                if (nextStage && pool < 5) {
                    shouldReset = true;
                    targetPool = pool + 1;
                    const previousProtectionCount = appState.protectionCards;
                    appState.protectionCards = 0;
                    if (previousProtectionCount > 0) {
                        showMessage("通过十号任务进入新阶段，保护卡已清零");
                    }
                } else if (jumpToPool && jumpToPool >= 1 && jumpToPool <= 5) {
                    if (appState.protectionCards > 0) {
                        appState.protectionCards--;
                        jumpPrevented = true;
                        showMessage("已自动使用保护卡，免疫了此次跳转");
                    } else {
                        targetPool = jumpToPool;
                        showMessage(`已跳转到阶段${jumpToPool}`);
                    }
                }
                
                // 保存重置状态，调整执行顺序
                const needReset = shouldReset;
                
                if (targetPool !== pool && !jumpPrevented) {
                    switchToPool(targetPool);
                    applyDartCardEffect(targetPool);
                    addRandomCat(targetPool);
                }
                
                // 如果是进入下一阶段，重置除当前阶段外的所有奖池的完成状态
                if (needReset) {
                    resetPoolsExceptCurrent(targetPool);
                }
                
                // 应用连弩卡效果（50%概率）
                if (appState.crossbowCardEnabled && Math.random() < 0.5) {
                    applyCrossbowCardEffect();
                }
                
                // 检查是否完成了第五阶段的十号任务（最终挑战）
                if (pool === 5 && index === 9) {
                    // 标记游戏为已完成
                    appState.gameCompleted = true;
                    // 关闭挑战模态框后显示通关界面
                    elements.closeChallengeModal.addEventListener('click', showGameClearScreen, { once: true });
                }
                
                showMessage(`挑战成功！${
                    catType === 'gentle' ? '温柔猫猫' : 
                    catType === 'bad' ? '坏蛋猫猫' : '狡猾猫猫'
                }已被安抚，获得额外${bonusCoins}枚金币`);
            }
            
            // 关闭模态框的事件会处理UI重置
        }

        // 逃避猫猫
        function escapeMonster() {
            if (!appState.lastDrawnOption) return;
            
            const { catType } = appState.lastDrawnOption;
            
            // 污染指数提高10%
            appState.pollutionIndex += 10;
            
            // 重置状态，但不标记任务为完成
            appState.lastDrawnOption = null;
            appState.lastStarTask = null;
            appState.taskInProgress = false;
            
            // 更新UI
            renderPrizeOptions();
            elements.drawResultEl.innerHTML = '<p class="text-gray-400">点击上方任务选项进行抽取</p>';
            elements.starTaskResultEl.innerHTML = '<p class="text-gray-400">抽取主任务后将获得额外挑战</p>';
            elements.completeTaskBtn.disabled = true;
            elements.attackMonsterBtn.classList.add('hidden');
            elements.escapeMonsterBtn.classList.add('hidden');
            elements.drawRandomBtn.disabled = false;
            updatePoolStatus();
            updateCounters();
            
            showMessage(`你选择了逃避，污染指数上升10%，${
                catType === 'gentle' ? '温柔猫猫' : 
                catType === 'bad' ? '坏蛋猫猫' : '狡猾猫猫'
            }仍然在附近`);
        }

        // 切换到指定奖池
        function switchToPool(poolNumber) {
            if (poolNumber < 1 || poolNumber > 5 || appState.gameCompleted) return;
            
            appState.currentPool = poolNumber;
            elements.currentPoolEl.textContent = `阶段 ${poolNumber}`;
            
            // 更新导航按钮样式
            elements.poolNavBtns.forEach(btn => {
                const btnPool = parseInt(btn.dataset.pool);
                if (btnPool === poolNumber) {
                    btn.className = 'pool-nav-btn bg-primary text-white rounded-lg p-3 font-bold transition-all';
                } else {
                    btn.className = 'pool-nav-btn bg-gray-200 hover:bg-gray-300 text-gray-700 rounded-lg p-3 font-bold transition-all';
                }
            });
            
            renderPrizeOptions();
        }

        // 显示完成任务动画
        function showCompletionAnimation() {
            const animationEl = document.createElement('div');
            animationEl.className = 'fixed top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-success text-white p-6 rounded-full z-50 flex items-center justify-center';
            animationEl.innerHTML = '<i class="fa fa-check text-5xl"></i>';
            document.body.appendChild(animationEl);
            
            // 添加动画效果
            setTimeout(() => {
                animationEl.classList.add('opacity-0', 'transition-opacity', 'duration-500');
                setTimeout(() => {
                    document.body.removeChild(animationEl);
                }, 500);
            }, 1000);
        }

        // 显示消息提示
        function showMessage(message) {
            const messageEl = document.createElement('div');
            messageEl.className = 'fixed top-20 left-1/2 transform -translate-x-1/2 bg-dark text-white px-4 py-2 rounded-lg z-50 flex items-center';
            messageEl.innerHTML = `<p>${message}</p>`;
            document.body.appendChild(messageEl);
            
            // 添加动画效果
            setTimeout(() => {
                messageEl.classList.add('opacity-0', 'transition-opacity', 'duration-500');
                setTimeout(() => {
                    document.body.removeChild(messageEl);
                }, 500);
            }, 2000);
        }

        // 显示通关成功界面
        function showGameClearScreen() {
            // 填充通关数据
            elements.finalTasksCountEl.textContent = appState.completedTasksCount;
            elements.finalGroupsCountEl.textContent = appState.completedGroupsCount;
            elements.finalCoinsCountEl.textContent = appState.coins;
            elements.finalPollutionIndexEl.textContent = `${appState.pollutionIndex}%`;
            
            // 显示通关模态框
            elements.gameClearModal.classList.remove('hidden');
            
            // 显示庆祝动画
            createConfetti();
            
            // 禁用所有交互按钮
            elements.shopBtn.disabled = true;
            elements.helpBtn.disabled = true;
            elements.editPoolBtn.disabled = true;
            elements.drawRandomBtn.disabled = true;
            elements.poolNavBtns.forEach(btn => btn.disabled = true);
            
            // 更新UI显示
            renderPrizeOptions();
        }

        // 创建庆祝彩屑动画
        function createConfetti() {
            const colors = ['#FF6B35', '#175676', '#4CC9F0', '#FFC107', '#F44336', '#FFD700', '#9C27B0', '#8BC34A'];
            const count = 100;
            
            for (let i = 0; i < count; i++) {
                const confetti = document.createElement('div');
                confetti.className = 'confetti';
                confetti.style.left = `${Math.random() * 100}vw`;
                confetti.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)];
                confetti.style.width = `${Math.random() * 10 + 5}px`;
                confetti.style.height = `${Math.random() * 10 + 5}px`;
                confetti.style.animationDelay = `${Math.random() * 5}s`;
                document.body.appendChild(confetti);
                
                // 动画结束后移除元素
                setTimeout(() => {
                    confetti.remove();
                }, 5000);
            }
        }

        // 重新开始游戏
        function restartGame() {
            // 重置应用状态
            appState.currentPool = 1;
            appState.pools = generatePools();
            appState.lastDrawnOption = null;
            appState.lastStarTask = null;
            appState.taskInProgress = false;
            appState.gameCompleted = false;
            appState.completedTasksCount = 0;
            appState.completedGroupsCount = 0;
            appState.coins = 0;
            appState.protectionCards = 0;
            appState.dartCardEnabled = false;
            // 新增道具重置
            appState.meowCardEnabled = false;
            appState.crossbowCardEnabled = false;
            appState.pollutionIndex = 20;
            
            // 隐藏通关模态框
            elements.gameClearModal.classList.add('hidden');
            
            // 启用所有交互按钮
            elements.shopBtn.disabled = false;
            elements.helpBtn.disabled = false;
            elements.editPoolBtn.disabled = false;
            elements.drawRandomBtn.disabled = false;
            elements.poolNavBtns.forEach(btn => btn.disabled = false);
            
            // 更新UI
            switchToPool(1);
            elements.drawResultEl.innerHTML = '<p class="text-gray-400">点击上方任务选项进行抽取</p>';
            elements.starTaskResultEl.innerHTML = '<p class="text-gray-400">抽取主任务后将获得额外挑战</p>';
            updateCounters();
            updatePoolStatus();
            
            // 清除所有彩屑元素
            document.querySelectorAll('.confetti').forEach(confetti => confetti.remove());
        }

        // 更新星桶任务数量
        function updateStarTasksCount() {
            elements.starTasksCountEl.textContent = appState.starTasks.length;
        }

        // 打开奖池编辑模态框
        function openEditPoolModal() {
            if (appState.gameCompleted) return;
            
            elements.editingPoolText.textContent = `阶段 ${appState.currentPool}`;
            renderEditOptions();
            elements.editPoolModal.classList.remove('hidden');
        }

        // 渲染编辑选项
        function renderEditOptions() {
            const currentOptions = appState.pools[appState.currentPool - 1];
            elements.editOptionsContainer.innerHTML = '';
            
            currentOptions.forEach((option, index) => {
                const optionEl = document.createElement('div');
                optionEl.className = 'border border-gray-200 rounded-xl p-4';
                optionEl.innerHTML = `
                    <div class="flex items-center mb-3">
                        <div class="w-8 h-8 bg-primary/10 text-primary rounded-full flex items-center justify-center font-bold mr-3">
                            ${index + 1}
                        </div>
                        <h4 class="font-medium">任务 ${index + 1}</h4>
                        ${option.catType === 'gentle' ? '<span class="ml-2 bg-cat/10 text-cat px-2 py-0.5 rounded text-xs">有温柔猫猫</span>' : ''}
                        ${option.catType === 'bad' ? '<span class="ml-2 bg-badcat/10 text-badcat px-2 py-0.5 rounded text-xs">有坏蛋猫猫</span>' : ''}
                        ${option.catType === 'cunning' ? '<span class="ml-2 bg-cunningcat/10 text-cunningcat px-2 py-0.5 rounded text-xs">有狡猾猫猫</span>' : ''}
                    </div>
                    
                    <div class="mb-3">
                        <label class="block text-sm text-gray-600 mb-1">任务内容</label>
                        <textarea class="option-text w-full border border-gray-300 rounded-lg p-2 text-sm" rows="2">${option.text}</textarea>
                    </div>
                    
                    <div class="flex items-center space-x-4">
                        <div class="flex items-center">
                            <input type="checkbox" class="option-next-stage mr-2" ${option.nextStage ? 'checked' : ''}>
                            <label class="text-sm text-gray-600">进入下一阶段</label>
                        </div>
                        
                        <div class="flex items-center">
                            <input type="checkbox" class="option-jump-enable mr-2" ${option.jumpToPool ? 'checked' : ''}>
                            <label class="text-sm text-gray-600 mr-2">跳转到阶段</label>
                            <select class="option-jump-to border border-gray-300 rounded text-sm px-2 py-1" ${!option.jumpToPool ? 'disabled' : ''}>
                                <option value="1" ${option.jumpToPool === 1 ? 'selected' : ''}>1</option>
                                <option value="2" ${option.jumpToPool === 2 ? 'selected' : ''}>2</option>
                                <option value="3" ${option.jumpToPool === 3 ? 'selected' : ''}>3</option>
                                <option value="4" ${option.jumpToPool === 4 ? 'selected' : ''}>4</option>
                                <option value="5" ${option.jumpToPool === 5 ? 'selected' : ''}>5</option>
                            </select>
                        </div>
                    </div>
                `;
                
                // 添加跳转复选框事件
                const jumpEnable = optionEl.querySelector('.option-jump-enable');
                const jumpTo = optionEl.querySelector('.option-jump-to');
                
                jumpEnable.addEventListener('change', () => {
                    jumpTo.disabled = !jumpEnable.checked;
                });
                
                elements.editOptionsContainer.appendChild(optionEl);
            });
        }

        // 保存奖池选项编辑
        function savePoolOptions() {
            const currentOptions = appState.pools[appState.currentPool - 1];
            const editElements = elements.editOptionsContainer.querySelectorAll('div[class*="border-gray-200"]');
            
            editElements.forEach((el, index) => {
                const text = el.querySelector('.option-text').value.trim() || `任务 ${index + 1}`;
                const nextStage = el.querySelector('.option-next-stage').checked;
                const jumpEnable = el.querySelector('.option-jump-enable').checked;
                const jumpToPool = jumpEnable ? parseInt(el.querySelector('.option-jump-to').value) : null;
                // 提取组数信息
                const groupCountMatch = text.match(/\d+/);
                const groupCount = groupCountMatch ? parseInt(groupCountMatch[0]) : 1;
                
                currentOptions[index] = {
                    ...currentOptions[index],
                    text,
                    nextStage,
                    jumpToPool,
                    groupCount: groupCount,
                    // 保留完成状态、完成方式和猫属性
                    completed: currentOptions[index].completed,
                    completedByDart: currentOptions[index].completedByDart,
                    catType: currentOptions[index].catType
                };
            });
            
            elements.editPoolModal.classList.add('hidden');
            renderPrizeOptions();
        }

        // 打开星桶任务编辑模态框
        function openEditStarModal() {
            if (appState.gameCompleted) return;
            
            renderStarTasks();
            elements.editStarModal.classList.remove('hidden');
        }

        // 渲染星桶任务编辑
        function renderStarTasks() {
            elements.starTasksContainer.innerHTML = '';
            
            appState.starTasks.forEach((task, index) => {
                const taskEl = document.createElement('div');
                taskEl.className = 'flex items-center space-x-3';
                taskEl.innerHTML = `
                    <div class="w-6 h-6 bg-warning/10 text-warning rounded-full flex items-center justify-center text-xs font-bold flex-shrink-0">
                        ${index + 1}
                    </div>
                    <input type="text" value="${task}" class="star-task-text flex-grow border border-gray-300 rounded-lg p-2 text-sm">
                    <button class="delete-star-task text-gray-400 hover:text-danger p-1" data-index="${index}">
                        <i class="fa fa-trash"></i>
                    </button>
                `;
                
                elements.starTasksContainer.appendChild(taskEl);
            });
            
            // 添加删除事件
            document.querySelectorAll('.delete-star-task').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    const index = parseInt(e.currentTarget.dataset.index);
                    appState.starTasks.splice(index, 1);
                    renderStarTasks();
                });
            });
        }

        // 添加新的星桶任务
        function addNewStarTask() {
            appState.starTasks.push(`新额外挑战 ${appState.starTasks.length + 1}`);
            renderStarTasks();
            
            // 自动聚焦到新任务输入框
            const inputs = document.querySelectorAll('.star-task-text');
            inputs[inputs.length - 1].focus();
        }

        // 保存星桶任务编辑
        function saveStarTasks() {
            const taskInputs = document.querySelectorAll('.star-task-text');
            appState.starTasks = Array.from(taskInputs).map(input => {
                return input.value.trim() || '未命名挑战';
            });
            
            elements.editStarModal.classList.add('hidden');
            updateStarTasksCount();
        }

        // 打开商店模态框
        function openShopModal() {
            if (appState.gameCompleted) return;
            
            updateCounters(); // 确保显示最新数据
            elements.shopModal.classList.remove('hidden');
        }

        // 购买保护卡
        function buyProtectionCard() {
            if (appState.coins >= appState.protectionCardPrice) {
                appState.coins -= appState.protectionCardPrice;
                appState.protectionCards++;
                updateCounters();
                showMessage("成功购买1张保护卡");
            }
        }

        // 购买并启用飞镖卡
        function buyDartCard() {
            if (appState.coins >= appState.dartCardPrice && !appState.dartCardEnabled) {
                appState.coins -= appState.dartCardPrice;
                appState.dartCardEnabled = true;
                updateCounters();
                showMessage("成功购买并启用飞镖卡");
            }
        }
        
        // 购买并启用喵喵卡
        function buyMeowCard() {
            if (appState.coins >= appState.meowCardPrice && !appState.meowCardEnabled) {
                appState.coins -= appState.meowCardPrice;
                appState.meowCardEnabled = true;
                updateCounters();
                showMessage("成功购买并启用喵喵卡");
            }
        }
        
        // 购买并启用连弩卡
        function buyCrossbowCard() {
            if (appState.coins >= appState.crossbowCardPrice && !appState.crossbowCardEnabled) {
                appState.coins -= appState.crossbowCardPrice;
                appState.crossbowCardEnabled = true;
                updateCounters();
                showMessage("成功购买并启用连弩卡");
            }
        }

        // 设置事件监听器
        function setupEventListeners() {
            // 随机抽取按钮
            elements.drawRandomBtn.addEventListener('click', drawRandomOption);
            
            // 完成任务按钮
            elements.completeTaskBtn.addEventListener('click', completeCurrentTask);
            
            // 猫猫相关按钮
            elements.attackMonsterBtn.addEventListener('click', attackMonster);
            elements.escapeMonsterBtn.addEventListener('click', escapeMonster);
            
            // 节拍器挑战模态框
            elements.closeChallengeModal.addEventListener('click', () => {
                elements.beatChallengeModal.classList.add('hidden');
                
                // 重置状态
                appState.lastDrawnOption = null;
                appState.lastStarTask = null;
                appState.taskInProgress = false;
                
                // 更新UI
                renderPrizeOptions();
                elements.drawResultEl.innerHTML = '<p class="text-gray-400">点击上方任务选项进行抽取</p>';
                elements.starTaskResultEl.innerHTML = '<p class="text-gray-400">抽取主任务后将获得额外挑战</p>';
                elements.drawRandomBtn.disabled = false;
                updatePoolStatus();
                updateCounters();
            });
            
            // 奖池导航按钮
            elements.poolNavBtns.forEach(btn => {
                btn.addEventListener('click', () => {
                    if (!appState.taskInProgress && !appState.gameCompleted) {
                        const poolNumber = parseInt(btn.dataset.pool);
                        // 手动切换阶段也应用飞镖卡效果
                        const needApplyDart = appState.currentPool !== poolNumber;
                        switchToPool(poolNumber);
                        
                        if (needApplyDart) {
                            applyDartCardEffect(poolNumber);
                            // 阶段跳转时随机添加猫属性
                            addRandomCat(poolNumber);
                        }
                    }
                });
            });
            
            // 编辑奖池选项
            elements.editPoolBtn.addEventListener('click', openEditPoolModal);
            elements.closePoolModal.addEventListener('click', () => {
                elements.editPoolModal.classList.add('hidden');
            });
            elements.savePoolOptions.addEventListener('click', savePoolOptions);
            
            // 商店相关
            elements.shopBtn.addEventListener('click', openShopModal);
            elements.closeShopModal.addEventListener('click', () => {
                elements.shopModal.classList.add('hidden');
            });
            elements.buyProtectionBtn.addEventListener('click', buyProtectionCard);
            elements.buyDartBtn.addEventListener('click', buyDartCard);
            elements.buyMeowBtn.addEventListener('click', buyMeowCard);
            elements.buyCrossbowBtn.addEventListener('click', buyCrossbowCard);
            
            // 帮助模态框
            elements.helpBtn.addEventListener('click', () => {
                elements.helpModal.classList.remove('hidden');
            });
            elements.closeHelpModal.addEventListener('click', () => {
                elements.helpModal.classList.add('hidden');
            });
            
            // 通关相关
            elements.restartGameBtn.addEventListener('click', restartGame);
            
            // 点击模态框外部关闭
            elements.editPoolModal.addEventListener('click', (e) => {
                if (e.target === elements.editPoolModal) {
                    elements.editPoolModal.classList.add('hidden');
                }
            });
            
            elements.helpModal.addEventListener('click', (e) => {
                if (e.target === elements.helpModal) {
                    elements.helpModal.classList.add('hidden');
                }
            });
            
            elements.shopModal.addEventListener('click', (e) => {
                if (e.target === elements.shopModal) {
                    elements.shopModal.classList.add('hidden');
                }
            });
            
            elements.beatChallengeModal.addEventListener('click', (e) => {
                if (e.target === elements.beatChallengeModal) {
                    // 不允许在挑战进行中关闭
                    if (!elements.closeChallengeModal.disabled) {
                        elements.beatChallengeModal.classList.add('hidden');
                        // 处理UI重置
                        renderPrizeOptions();
                        elements.drawResultEl.innerHTML = '<p class="text-gray-400">点击上方任务选项进行抽取</p>';
                        elements.starTaskResultEl.innerHTML = '<p class="text-gray-400">抽取主任务后将获得额外挑战</p>';
                        elements.drawRandomBtn.disabled = false;
                        updatePoolStatus();
                        updateCounters();
                    }
                }
            });
            
            elements.gameClearModal.addEventListener('click', (e) => {
                if (e.target === elements.gameClearModal) {
                    elements.gameClearModal.classList.add('hidden');
                }
            });
            
            // 滚动时改变导航栏样式
            window.addEventListener('scroll', () => {
                if (window.scrollY > 10) {
                    elements.mainHeader.classList.add('py-2', 'shadow-lg');
                    elements.mainHeader.classList.remove('py-3', 'shadow-md');
                } else {
                    elements.mainHeader.classList.add('py-3', 'shadow-md');
                    elements.mainHeader.classList.remove('py-2', 'shadow-lg');
                }
            });
        }

        // 启动应用
        document.addEventListener('DOMContentLoaded', initApp);
    </script>
</body>
</html>
