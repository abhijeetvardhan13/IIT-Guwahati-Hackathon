<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>NutriNode | AI-Native Health Scanner</title>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #0f172a; color: white; -webkit-tap-highlight-color: transparent; }
        .glass-panel { background: rgba(30, 41, 59, 0.7); backdrop-filter: blur(12px); border: 1px solid rgba(255, 255, 255, 0.1); }
        .ai-gradient-text { background: linear-gradient(135deg, #60a5fa 0%, #a78bfa 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .typing-cursor::after { content: '|'; animation: blink 1s step-start infinite; }
        @keyframes blink { 50% { opacity: 0; } }
        .scan-line { height: 2px; background: #60a5fa; box-shadow: 0 0 10px #60a5fa; animation: scan 2s linear infinite; }
        @keyframes scan { 0% { top: 0; opacity: 0; } 10% { opacity: 1; } 90% { opacity: 1; } 100% { top: 100%; opacity: 0; } }
        /* Hide scrollbar for clean UI */
        ::-webkit-scrollbar { width: 0px; background: transparent; }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;
        
        // --- MOCK DATA FOR SIMULATION ---
        // In a real app, this would come from OpenFoodFacts + Thesys Reasoning
        const PRODUCT_SCENARIOS = {
            "soda": {
                name: "TurboFizz Energy Cola",
                image: "https://images.unsplash.com/photo-1622483767028-3f66f32aef97?auto=format&fit=crop&q=80&w=300&h=300",
                intent: "Energy Boost",
                riskLevel: "high",
                healthScore: 35,
                tags: ["High Caffeine", "Added Sugar", "Artificial Color"],
                analysis: [
                    { type: "danger", title: "Sugar Crash Imminent", content: "Contains 45g of High Fructose Corn Syrup. While this provides instant energy, it exceeds your daily limit (36g) by 125%, likely leading to a crash within 90 minutes." },
                    { type: "info", title: "Caffeine Content", content: "120mg caffeine is equivalent to 1.5 cups of coffee. Good for acute focus, bad for sleep if consumed now (8 PM)." },
                    { type: "recommendation", title: "Better Alternative", content: "Try 'SparkleWater Citrus' for the fizz without the glycemic spike." }
                ]
            },
            "yogurt": {
                name: "Oat-Lyfe Probiotic Yogurt",
                image: "https://images.unsplash.com/photo-1563379926898-05f4575a45d8?auto=format&fit=crop&q=80&w=300&h=300",
                intent: "Gut Health",
                riskLevel: "low",
                healthScore: 92,
                tags: ["Probiotic", "Vegan", "Low Sugar"],
                analysis: [
                    { type: "success", title: "Excellent Choice", content: "Contains Lactobacillus rhamnosus, which aligns with your goal of improving gut microbiome diversity." },
                    { type: "info", title: "Nutrient Density", content: "Provides 12g of plant-based protein per serving, covering 20% of your daily needs." }
                ]
            },
            "cereal": {
                name: "Morning Crunch Puffs",
                image: "https://images.unsplash.com/photo-1521483450569-b5f8a47103fa?auto=format&fit=crop&q=80&w=300&h=300",
                intent: "Quick Breakfast",
                riskLevel: "medium",
                healthScore: 60,
                tags: ["Fortified", "Processed", "Fiber"],
                analysis: [
                    { type: "warning", title: "Hidden Sodium", content: "While marketed as healthy, one bowl contains 300mg of sodium. Be mindful if you have hypertension concerns." },
                    { type: "success", title: "Iron Fortified", content: "Good source of iron (18mg), helpful for your reported anemia history." }
                ]
            }
        };

        // --- COMPONENTS ---

        const Icon = ({ name, size = 20, className = "" }) => {
            return <i data-lucide={name} width={size} height={size} className={className}></i>;
        };

        const NavBar = () => (
            <div className="fixed bottom-0 left-0 w-full glass-panel border-t border-slate-700 p-4 flex justify-around items-center z-50 pb-6">
                <button className="text-slate-400 hover:text-blue-400 flex flex-col items-center">
                    <Icon name="home" />
                    <span className="text-xs mt-1">Home</span>
                </button>
                <button className="bg-blue-600 hover:bg-blue-500 text-white p-4 rounded-full -mt-8 shadow-lg shadow-blue-500/30 flex items-center justify-center">
                    <Icon name="scan-line" size={28} />
                </button>
                <button className="text-slate-400 hover:text-blue-400 flex flex-col items-center">
                    <Icon name="user" />
                    <span className="text-xs mt-1">Profile</span>
                </button>
            </div>
        );

        const AnalysisCard = ({ type, title, content, delay }) => {
            const [visible, setVisible] = useState(false);
            
            useEffect(() => {
                const timer = setTimeout(() => setVisible(true), delay);
                return () => clearTimeout(timer);
            }, [delay]);

            if (!visible) return null;

            const styles = {
                danger: "border-red-500/50 bg-red-950/20 text-red-200",
                warning: "border-yellow-500/50 bg-yellow-950/20 text-yellow-200",
                success: "border-green-500/50 bg-green-950/20 text-green-200",
                info: "border-blue-500/50 bg-blue-950/20 text-blue-200",
                recommendation: "border-purple-500/50 bg-purple-950/20 text-purple-200"
            };

            const icons = {
                danger: "alert-octagon",
                warning: "alert-triangle",
                success: "check-circle-2",
                info: "info",
                recommendation: "sparkles"
            };

            return (
                <div className={`mb-4 p-4 rounded-xl border ${styles[type]} transform transition-all duration-500 translate-y-0 opacity-100 animate-fade-in-up`}>
                    <div className="flex items-start gap-3">
                        <div className="mt-1"><Icon name={icons[type]} /></div>
                        <div>
                            <h3 className="font-bold text-sm uppercase tracking-wider opacity-80 mb-1">{title}</h3>
                            <p className="text-sm leading-relaxed opacity-90">{content}</p>
                        </div>
                    </div>
                </div>
            );
        };

        const ScoreGauge = ({ score }) => {
            const getColor = (s) => s > 80 ? 'text-green-400' : s > 50 ? 'text-yellow-400' : 'text-red-400';
            const getLabel = (s) => s > 80 ? 'Excellent' : s > 50 ? 'Moderate' : 'Avoid';
            
            return (
                <div className="flex flex-col items-center justify-center p-6 glass-panel rounded-2xl mb-6">
                    <div className="relative w-32 h-32 flex items-center justify-center">
                        <svg className="w-full h-full transform -rotate-90">
                            <circle cx="64" cy="64" r="56" stroke="currentColor" strokeWidth="8" fill="transparent" className="text-slate-700" />
                            <circle cx="64" cy="64" r="56" stroke="currentColor" strokeWidth="8" fill="transparent" 
                                className={`${getColor(score)} transition-all duration-1000 ease-out`}
                                strokeDasharray={351}
                                strokeDashoffset={351 - (351 * score / 100)}
                            />
                        </svg>
                        <div className="absolute inset-0 flex flex-col items-center justify-center">
                            <span className={`text-4xl font-bold ${getColor(score)}`}>{score}</span>
                            <span className="text-xs text-slate-400 uppercase tracking-widest mt-1">Score</span>
                        </div>
                    </div>
                    <div className={`mt-2 font-medium ${getColor(score)}`}>{getLabel(score)} Match</div>
                    <p className="text-xs text-slate-400 text-center mt-2 max-w-[200px]">Based on your goal: <strong>"Reduce Inflammation"</strong></p>
                </div>
            );
        };

        // --- MAIN APP ---

        const App = () => {
            const [view, setView] = useState('home'); // home, scanning, analyzing, result
            const [selectedProduct, setSelectedProduct] = useState(null);
            
            useEffect(() => {
                lucide.createIcons();
            }, [view]);

            const startScan = (productKey) => {
                setSelectedProduct(PRODUCT_SCENARIOS[productKey]);
                setView('scanning');
                setTimeout(() => setView('analyzing'), 2000);
                setTimeout(() => setView('result'), 4500);
            };

            // VIEW: HOME
            if (view === 'home') {
                return (
                    <div className="min-h-screen pb-24 p-6 flex flex-col">
                        <header className="flex justify-between items-center mb-8 pt-4">
                            <div className="flex items-center gap-2">
                                <div className="w-8 h-8 bg-gradient-to-tr from-blue-500 to-purple-500 rounded-lg flex items-center justify-center">
                                    <Icon name="brain-circuit" size={18} className="text-white" />
                                </div>
                                <span className="font-bold text-xl tracking-tight">NutriNode</span>
                            </div>
                            <div className="w-10 h-10 rounded-full bg-slate-700 flex items-center justify-center border border-slate-600">
                                <span className="text-xs font-bold">JD</span>
                            </div>
                        </header>

                        <div className="mb-8">
                            <h1 className="text-3xl font-light mb-2">Hello, <span className="font-bold text-white">John</span></h1>
                            <p className="text-slate-400">Your goal: <span className="text-blue-400">Reduce Inflammation</span></p>
                        </div>

                        <div className="flex-1 flex flex-col justify-center items-center">
                            <div className="w-full glass-panel rounded-2xl p-8 mb-8 text-center border-dashed border-2 border-slate-600 hover:border-blue-500 transition-colors cursor-pointer group">
                                <div className="w-16 h-16 bg-slate-800 rounded-full flex items-center justify-center mx-auto mb-4 group-hover:bg-blue-900/50 transition-colors">
                                    <Icon name="camera" className="text-blue-400" size={32} />
                                </div>
                                <h3 className="text-lg font-semibold mb-1">Scan a Product</h3>
                                <p className="text-slate-400 text-sm">Point at any barcode or label</p>
                            </div>

                            <p className="text-xs uppercase tracking-widest text-slate-500 mb-4">Or try a demo scan</p>
                            
                            <div className="grid grid-cols-3 gap-3 w-full">
                                {Object.entries(PRODUCT_SCENARIOS).map(([key, item]) => (
                                    <button 
                                        key={key}
                                        onClick={() => startScan(key)}
                                        className="glass-panel p-3 rounded-xl flex flex-col items-center gap-2 hover:bg-slate-700 transition-colors"
                                    >
                                        <img src={item.image} className="w-10 h-10 rounded-full object-cover" />
                                        <span className="text-[10px] text-center opacity-70 leading-tight">{item.name}</span>
                                    </button>
                                ))}
                            </div>
                        </div>
                        <NavBar />
                    </div>
                );
            }

            // VIEW: SCANNING
            if (view === 'scanning') {
                return (
                    <div className="min-h-screen bg-black relative flex flex-col items-center justify-center overflow-hidden">
                        {/* Mock Camera Feed Background */}
                        <div className="absolute inset-0 opacity-50 bg-[url('https://images.unsplash.com/photo-1542838132-92c53300491e?auto=format&fit=crop&q=80&w=1000')] bg-cover bg-center filter blur-sm"></div>
                        
                        <div className="relative z-10 w-64 h-64 border-2 border-white/30 rounded-3xl flex items-center justify-center overflow-hidden">
                            <div className="absolute top-0 left-0 w-full h-full border-4 border-blue-500/50 rounded-3xl"></div>
                            <div className="absolute top-0 left-0 w-full h-1 bg-blue-400 scan-line shadow-[0_0_15px_rgba(96,165,250,0.8)]"></div>
                            <img src={selectedProduct.image} className="w-48 h-48 object-contain rounded-lg shadow-2xl" />
                        </div>
                        
                        <div className="absolute bottom-32 z-20 text-center">
                            <p className="text-lg font-medium animate-pulse">Scanning Label...</p>
                            <p className="text-sm text-slate-400">Keep product steady</p>
                        </div>
                        
                        <button onClick={() => setView('home')} className="absolute top-8 right-6 z-50 bg-black/50 p-2 rounded-full">
                            <Icon name="x" />
                        </button>
                    </div>
                );
            }

            // VIEW: ANALYZING (Generative UI Simulation)
            if (view === 'analyzing') {
                return (
                    <div className="min-h-screen flex flex-col items-center justify-center p-8 bg-[#0f172a]">
                        <div className="mb-8 relative">
                            <div className="w-20 h-20 bg-blue-500/20 rounded-full flex items-center justify-center animate-ping absolute"></div>
                            <div className="w-20 h-20 bg-gradient-to-tr from-blue-600 to-purple-600 rounded-full flex items-center justify-center relative z-10 shadow-xl shadow-blue-500/20">
                                <Icon name="sparkles" className="text-white animate-spin-slow" />
                            </div>
                        </div>
                        
                        <h2 className="text-2xl font-light mb-2 ai-gradient-text">NutriNode AI</h2>
                        <div className="text-slate-400 text-sm font-mono h-20 overflow-hidden text-center">
                            <p className="animate-pulse mb-1">Identifying ingredients...</p>
                            <p className="animate-pulse delay-100 mb-1">Accessing OpenFoodFacts DB...</p>
                            <p className="animate-pulse delay-200 mb-1">Contextualizing for inflammation...</p>
                            <p className="animate-pulse delay-300 text-blue-400">Generating UI components...</p>
                        </div>
                    </div>
                );
            }

            // VIEW: RESULT (The "AI-Native" Interface)
            if (view === 'result') {
                return (
                    <div className="min-h-screen bg-[#0f172a] pb-24">
                         <div className="sticky top-0 z-40 bg-[#0f172a]/90 backdrop-blur-md border-b border-white/10 p-4 flex items-center justify-between">
                            <button onClick={() => setView('home')} className="p-2 -ml-2 hover:bg-white/10 rounded-full"><Icon name="arrow-left" /></button>
                            <span className="font-semibold">{selectedProduct.name}</span>
                            <button className="p-2 -mr-2 hover:bg-white/10 rounded-full"><Icon name="share-2" /></button>
                        </div>

                        <div className="p-6">
                            {/* 1. Contextual Score */}
                            <ScoreGauge score={selectedProduct.healthScore} />

                            {/* 2. AI Reasoning Cards (Generative UI) */}
                            <div className="mb-6">
                                <h3 className="text-xs uppercase tracking-widest text-slate-500 mb-3 ml-1">AI Reasoning Engine</h3>
                                {selectedProduct.analysis.map((item, idx) => (
                                    <AnalysisCard 
                                        key={idx} 
                                        type={item.type} 
                                        title={item.title} 
                                        content={item.content} 
                                        delay={idx * 300} 
                                    />
                                ))}
                            </div>

                            {/* 3. Data Tags */}
                            <div className="flex flex-wrap gap-2 mb-8">
                                {selectedProduct.tags.map(tag => (
                                    <span key={tag} className="px-3 py-1 rounded-full bg-slate-800 border border-slate-700 text-xs text-slate-300">
                                        #{tag}
                                    </span>
                                ))}
                            </div>
                        </div>

                        <div className="fixed bottom-0 left-0 w-full p-4 bg-gradient-to-t from-[#0f172a] to-transparent z-30">
                            <button onClick={() => setView('home')} className="w-full bg-slate-800 hover:bg-slate-700 text-white font-medium py-4 rounded-xl shadow-lg border border-slate-700 transition-all flex items-center justify-center gap-2">
                                <Icon name="scan" size={18} />
                                Scan Another Item
                            </button>
                        </div>
                    </div>
                );
            }
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
