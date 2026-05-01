<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2muong Photoboost coquette</title>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Dancing+Script:wght@700&family=Pacifico&family=Montserrat:wght@900&family=Quicksand:wght@700&family=Playfair+Display:ital,wght@1,700&display=swap" rel="stylesheet">
    <style>
        body { background-color: #FFF5F7; margin: 0; font-family: 'Quicksand', sans-serif; overflow-x: hidden; }
        .clip-path-heart { clip-path: path('M50 15 C20 -5 0 25 0 55 C0 85 30 100 50 100 C70 100 100 85 100 55 C100 25 80 -5 50 15'); }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        .camera-overlay { background: radial-gradient(circle, transparent 40%, rgba(0,0,0,0.4) 100%); }
        /* Đảm bảo ảnh không bị méo khi render */
        .photo-container img { object-fit: cover; width: 100%; height: 100%; display: block; }
    </style>
</head>
<body>
    <div id="root"></div>

    <!-- {% raw %} -->
    <script type="text/babel">
        const { useState, useRef, useEffect } = React;

        const Icon = ({ name, size = 24, className = "" }) => {
            const icons = {
                camera: "📸", download: "💾", x: "✕", left: "←", sparkles: "✨", 
                refresh: "🔄", trash: "🗑️", type: "T", sticker: "🎀", layers: "🗂️", palette: "🎨", mirror: "🪞"
            };
            return <span className={className} style={{ fontSize: size, lineHeight: 1 }}>{icons[name] || '•'}</span>;
        };

        function App() {
            const [screen, setScreen] = useState('home'); 
            const [photos, setPhotos] = useState([]);
            const [layoutType, setLayoutType] = useState(4); 
            const [activeFrame, setActiveFrame] = useState(0);
            const [stickers, setStickers] = useState([]);
            const [isSaving, setIsSaving] = useState(false);
            const [countdown, setCountdown] = useState(null);
            const [activeFilter, setActiveFilter] = useState('none');
            const [beautyLevel, setBeautyLevel] = useState(true);
            const [isMirrored, setIsMirrored] = useState(true);
            const [photoShape, setPhotoShape] = useState('rounded'); 
            const [caption, setCaption] = useState('Coquette Love');
            const [activeFont, setActiveFont] = useState('Pacifico');

            const editorRef = useRef(null);
            const videoRef = useRef(null);
            const canvasRef = useRef(null);

            const filters = {
                none: { name: 'Gốc', style: '' },
                vibrant: { name: 'Xinh', style: 'saturate(1.3) brightness(1.05) contrast(1.02)' },
                pinky: { name: 'Hồng', style: 'sepia(0.1) hue-rotate(-10deg) saturate(1.4) brightness(1.1) contrast(0.95)' },
                warm: { name: 'Nắng', style: 'sepia(0.2) saturate(1.2) brightness(1.05)' },
                vintage: { name: 'Phim', style: 'sepia(0.4) contrast(1.1) brightness(0.9)' },
                bw: { name: 'B&W', style: 'grayscale(1) contrast(1.2)' }
            };

            const frames = [
                { id: 'white', name: 'Trắng', style: { background: '#FFFFFF' } },
                { id: 'pink-soft', name: 'Hồng Kem', style: { background: '#FFF0F3' } },
                { id: 'pink-pastel', name: 'Hồng Phấn', style: { background: '#FFD1DC' } },
                { id: 'blue-soft', name: 'Xanh Mây', style: { background: '#D0E1F9' } },
                { id: 'cream', name: 'Kem', style: { background: '#FFFDD0' } },
                { id: 'lavender', name: 'Oải Hương', style: { background: '#E6E6FA' } },
                { id: 'dark', name: 'Đen', style: { background: '#1A1A1A' } },
                { id: 'grad-pink', name: 'Tím Hồng', style: { background: 'linear-gradient(180deg, #FFDEE9 0%, #B5FFFC 100%)' } },
                { id: 'grad-sunset', name: 'Hoàng Hôn', style: { background: 'linear-gradient(45deg, #FF9A9E 0%, #FAD0C4 99%, #FAD0C4 100%)' } },
                { id: 'grid-pink', name: 'Caro Hồng', style: { background: '#FFF0F5', backgroundImage: 'repeating-linear-gradient(45deg, #FFB6C122 25%, transparent 25%, transparent 75%, #FFB6C122 75%, #FFB6C122), repeating-linear-gradient(45deg, #FFB6C122 25%, #FFF0F5 25%, #FFF0F5 75%, #FFB6C122 75%, #FFB6C122)', backgroundSize: '16px 16px' } },
                { id: 'dots', name: 'Chấm Bi', style: { background: '#FFFFFF', backgroundImage: 'radial-gradient(#FFB6C1 10%, transparent 10%)', backgroundSize: '20px 20px' } },
                { id: 'heart-pattern', name: 'Tim Bay', style: { background: '#FFF5F7', backgroundImage: 'url("data:image/svg+xml,%3Csvg width=\'20\' height=\'20\' viewBox=\'0 0 20 20\' xmlns=\'http://www.w3.org/2000/svg\'%3E%3Cpath d=\'M10 18l-1-1C4 13 1 10 1 6a4 4 0 0 1 7-3 4 4 0 0 1 7 3c0 4-3 7-8 11l-1 1z\' fill=\'%23FFB6C1\' fill-opacity=\'0.2\'/%3E%3C/svg%3E")' } },
                { id: 'check-black', name: 'Caro Đen', style: { background: '#FFFFFF', backgroundImage: 'conic-gradient(#000 0.25turn, #FFF 0.25turn 0.5turn, #000 0.5turn 0.75turn, #FFF 0.75turn)', backgroundSize: '30px 30px' } }
            ];

            const stickerAssets = ['🎀', '🐱', '🧸', '✨', '🍓', '🍰', '🐰', '💕', '🍒', '🦋', '🦢', '🕯️'];
            const shapes = { rect: 'rounded-none', rounded: 'rounded-[1rem]', circle: 'rounded-full aspect-square', heart: 'clip-path-heart' };
            const fonts = ['Pacifico', 'Dancing Script', 'Playfair Display', 'Montserrat'];

            const startPhotobooth = async () => {
                setPhotos([]);
                setScreen('camera');
                try {
                    const stream = await navigator.mediaDevices.getUserMedia({ 
                        video: { facingMode: 'user', width: { ideal: 1280 }, height: { ideal: 720 } } 
                    });
                    if (videoRef.current) videoRef.current.srcObject = stream;
                } catch (err) {
                    alert("Không thể truy cập Camera.");
                    setScreen('home');
                }
            };

            const captureSequence = async () => {
                const numToCapture = layoutType === 'filmstrip' ? 4 : (layoutType === 'polaroid' ? 1 : layoutType);
                const newPhotos = [];
                for (let i = 0; i < numToCapture; i++) {
                    for (let c = 3; c > 0; c--) {
                        setCountdown(c);
                        await new Promise(r => setTimeout(r, 1000));
                    }
                    setCountdown('📸');
                    const video = videoRef.current;
                    const canvas = canvasRef.current;
                    if (video && canvas) {
                        const size = Math.min(video.videoWidth, video.videoHeight);
                        canvas.width = size;
                        canvas.height = size;
                        
                        const ctx = canvas.getContext('2d');
                        ctx.save();
                        
                        if (isMirrored) {
                            ctx.translate(canvas.width, 0);
                            ctx.scale(-1, 1);
                        }

                        const startX = (video.videoWidth - size) / 2;
                        const startY = (video.videoHeight - size) / 2;
                        
                        ctx.filter = `${beautyLevel ? 'blur(0.5px) brightness(1.05)' : ''} ${filters[activeFilter].style}`;
                        ctx.drawImage(video, startX, startY, size, size, 0, 0, size, size);
                        ctx.restore();
                        
                        newPhotos.push(canvas.toDataURL('image/png'));
                    }
                    await new Promise(r => setTimeout(r, 600)); 
                    setCountdown(null);
                }
                setPhotos(newPhotos);
                if (videoRef.current?.srcObject) {
                    videoRef.current.srcObject.getTracks().forEach(t => t.stop());
                }
                setScreen('editor');
            };

            const downloadImage = async () => {
                if (isSaving) return;
                setIsSaving(true);
                await new Promise(r => setTimeout(r, 100));
                
                try {
                    const canvas = await html2canvas(editorRef.current, { 
                        scale: 3, 
                        useCORS: true,
                        backgroundColor: null,
                        logging: false
                    });
                    const link = document.createElement('a');
                    link.href = canvas.toDataURL("image/png");
                    link.download = `2muong-coquette-${Date.now()}.png`;
                    link.click();
                } catch (e) {
                    console.error(e);
                }
                setIsSaving(false);
            };

            if (screen === 'home') return (
                <div className="flex flex-col h-screen items-center justify-center p-8 text-center bg-[#FFF5F7]">
                    <div className="mb-10 relative">
                        <span className="absolute -top-10 -right-8 text-5xl rotate-12">🎀</span>
                        <h1 className="text-5xl font-black italic text-[#D44D5C] font-['Playfair_Display']">2muong Photoboost</h1>
                        <p className="text-[10px] tracking-[0.5em] uppercase font-bold text-[#FFB6C1] mt-2">Premium Coquette Edition</p>
                    </div>
                    
                    <div className="grid grid-cols-2 gap-4 mb-10 w-full max-w-xs bg-white/60 p-5 rounded-[2.5rem] shadow-sm border border-pink-100">
                        {[1, 2, 4, 'filmstrip', 'polaroid'].map(type => (
                            <button key={type} onClick={() => setLayoutType(type)} 
                                className={`p-4 rounded-2xl font-bold uppercase text-[10px] transition-all ${layoutType === type ? 'bg-[#D44D5C] text-white shadow-lg scale-105' : 'bg-white text-[#D44D5C] hover:bg-pink-50'}`}>
                                {type === 'filmstrip' ? '4 Dọc' : type === 'polaroid' ? '1 Polaroid' : `${type} Ảnh`}
                            </button>
                        ))}
                    </div>
                    <button onClick={startPhotobooth} className="bg-[#D44D5C] text-white px-16 py-6 rounded-full font-black shadow-2xl hover:scale-105 active:scale-95 transition-all text-lg italic">BẮT ĐẦU CHỤP</button>
                </div>
            );

            if (screen === 'camera') return (
                <div className="flex flex-col h-screen bg-black relative overflow-hidden">
                    <video ref={videoRef} autoPlay playsInline className="w-full h-full object-cover" 
                        style={{ transform: isMirrored ? 'scaleX(-1)' : 'scaleX(1)', filter: filters[activeFilter].style + (beautyLevel ? ' blur(0.3px)' : '') }} />
                    <canvas ref={canvasRef} className="hidden" />
                    <div className="absolute inset-0 camera-overlay pointer-events-none" />
                    {countdown && <div className="absolute inset-0 flex items-center justify-center text-white text-[10rem] font-black italic animate-pulse z-50">{countdown}</div>}
                    
                    <div className="absolute top-10 w-full px-6 flex flex-col gap-4">
                        <div className="flex justify-between items-center">
                            <button onClick={() => {
                                if (videoRef.current?.srcObject) videoRef.current.srcObject.getTracks().forEach(t => t.stop());
                                setScreen('home');
                            }} className="bg-white/20 backdrop-blur-md p-4 rounded-full text-white"><Icon name="x" /></button>
                            <div className="flex gap-3">
                                <button onClick={() => setIsMirrored(!isMirrored)} className={`p-4 rounded-full transition-all ${isMirrored ? 'bg-[#D44D5C] text-white' : 'bg-white/20 text-white'}`}><Icon name="mirror" /></button>
                                <button onClick={() => setBeautyLevel(!beautyLevel)} className={`p-4 rounded-full transition-all ${beautyLevel ? 'bg-[#FFB6C1] text-white' : 'bg-white/20 text-white'}`}><Icon name="sparkles" /></button>
                            </div>
                        </div>
                        <div className="flex gap-2 overflow-x-auto no-scrollbar py-2">
                            {Object.keys(filters).map(f => (
                                <button key={f} onClick={() => setActiveFilter(f)} 
                                    className={`px-6 py-2 rounded-full text-[11px] font-bold uppercase whitespace-nowrap border-2 transition-all ${activeFilter === f ? 'bg-white text-black border-white shadow-lg' : 'bg-black/30 text-white border-white/30'}`}>
                                    {filters[f].name}
                                </button>
                            ))}
                        </div>
                    </div>

                    <div className="absolute bottom-16 w-full flex justify-center">
                        <button onClick={captureSequence} disabled={countdown !== null}
                            className="w-24 h-24 bg-white rounded-full border-8 border-white/20 flex items-center justify-center active:scale-90 transition-all">
                            <div className="w-10 h-10 bg-[#D44D5C] rounded-full" />
                        </button>
                    </div>
                </div>
            );

            return (
                <div className="flex flex-col min-h-screen overflow-y-auto items-center p-6 bg-[#FFF5F7]">
                    <div className="w-full max-w-lg flex justify-between mb-8 sticky top-0 z-50 bg-[#FFF5F7]/80 backdrop-blur-md py-4">
                        <button onClick={() => setScreen('camera')} className="bg-white p-4 rounded-full shadow-md text-[#D44D5C]"><Icon name="left" /></button>
                        <button onClick={downloadImage} disabled={isSaving} 
                            className="bg-[#D44D5C] text-white px-8 rounded-full font-black flex items-center gap-2 shadow-xl active:scale-95 transition-all disabled:opacity-50">
                            {isSaving ? 'ĐANG LƯU...' : 'LƯU ẢNH'}
                        </button>
                    </div>

                    <div ref={editorRef} style={frames[activeFrame].style} 
                        className={`relative shadow-2xl flex flex-col items-center transition-all duration-300 overflow-hidden
                        ${layoutType === 'filmstrip' ? 'w-[240px] p-4 pt-6 pb-12' : 
                          layoutType === 'polaroid' ? 'w-[320px] p-6 pb-20 bg-white' : 
                          layoutType === 1 ? 'w-[320px] p-5 pb-10' :
                          'w-[340px] p-5 pb-14'}`}>
                        
                        <div className={`w-full grid gap-3 ${layoutType === 4 ? 'grid-cols-2' : 'grid-cols-1'}`}>
                            {photos.map((src, i) => (
                                <div key={i} className={`photo-container overflow-hidden relative shadow-sm ${shapes[photoShape]} aspect-square`}>
                                    <img src={src} />
                                </div>
                            ))}
                        </div>
                        
                        <div className="mt-6 text-center w-full relative z-10">
                            <p style={{ fontFamily: activeFont }} className="text-[#D44D5C] text-2xl px-2">{caption}</p>
                            <p className="text-[7px] opacity-40 font-bold mt-2 uppercase tracking-[0.4em]">© 2muong Photoboost</p>
                        </div>

                        {stickers.map(s => (
                            <div key={s.id} 
                                style={{ left: `${s.x}%`, top: `${s.y}%`, transform: 'translate(-50%, -50%)', fontSize: `${s.size}px` }}
                                className="absolute z-30 cursor-move select-none touch-none"
                                onMouseDown={(e) => {
                                    const rect = editorRef.current.getBoundingClientRect();
                                    const move = (me) => {
                                        const clientX = me.clientX || (me.touches?.[0].clientX);
                                        const clientY = me.clientY || (me.touches?.[0].clientY);
                                        setStickers(prev => prev.map(st => st.id === s.id ? {...st, x: (clientX - rect.left) / rect.width * 100, y: (clientY - rect.top) / rect.height * 100} : st));
                                    };
                                    const stop = () => { window.removeEventListener('mousemove', move); window.removeEventListener('mouseup', stop); window.removeEventListener('touchmove', move); window.removeEventListener('touchend', stop); };
                                    window.addEventListener('mousemove', move); window.addEventListener('mouseup', stop);
                                    window.addEventListener('touchmove', move); window.addEventListener('touchend', stop);
                                }}>
                                <div className="drop-shadow-md active:scale-125 transition-transform">{s.content}</div>
                            </div>
                        ))}
                    </div>

                    <div className="mt-12 w-full max-w-md bg-white p-8 rounded-[3rem] space-y-8 shadow-xl border border-pink-50 mb-20">
                        <div className="space-y-4">
                            <h3 className="text-[10px] font-black uppercase text-[#D44D5C] tracking-widest flex items-center gap-2"><Icon name="sticker" size={14}/> Stickers</h3>
                            <div className="grid grid-cols-4 gap-3">
                                {stickerAssets.map(s => <button key={s} onClick={() => setStickers([...stickers, { id: Date.now(), content: s, x: 50, y: 50, size: 50 }])} className="text-3xl p-3 bg-pink-50 hover:bg-pink-100 rounded-2xl active:scale-90">{s}</button>)}
                            </div>
                            <button onClick={() => setStickers([])} className="text-[9px] font-bold text-gray-400 uppercase w-full py-2 bg-gray-50 rounded-xl">Xóa sticker</button>
                        </div>

                        <div className="space-y-4">
                            <h3 className="text-[10px] font-black uppercase text-[#D44D5C] tracking-widest flex items-center gap-2"><Icon name="palette" size={14}/> Nền khung</h3>
                            <div className="flex gap-3 overflow-x-auto no-scrollbar py-2">
                                {frames.map((f, i) => (
                                    <button key={f.id} onClick={() => setActiveFrame(i)} 
                                        className={`w-12 h-12 rounded-xl border-2 flex-shrink-0 ${activeFrame === i ? 'border-[#D44D5C] scale-110 shadow-md' : 'border-gray-100'}`} style={f.style} />
                                ))}
                            </div>
                        </div>

                        <div className="space-y-4">
                            <h3 className="text-[10px] font-black uppercase text-[#D44D5C] tracking-widest flex items-center gap-2"><Icon name="type" size={14}/> Lời nhắn</h3>
                            <input className="w-full p-4 bg-pink-50 rounded-[1.5rem] text-center font-bold text-[#D44D5C]" value={caption} onChange={e => setCaption(e.target.value)} />
                            <div className="flex gap-2 overflow-x-auto no-scrollbar">
                                {fonts.map(f => <button key={f} onClick={() => setActiveFont(f)} style={{ fontFamily: f }} className={`px-4 py-2 rounded-full whitespace-nowrap border ${activeFont === f ? 'bg-[#D44D5C] text-white' : 'bg-white text-gray-400'}`}>{f}</button>)}
                            </div>
                        </div>
                    </div>
                </div>
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
    <!-- {% endraw %} -->
</body>
</html>
