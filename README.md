local a=game:GetService("Players")local b=game:GetService("RunService")local c=game:GetService("UserInputService")
local d=game:GetService("TweenService")local e=game:GetService("Workspace")local f=game:GetService("ReplicatedStorage")
local g=game:GetService("HttpService")local h=a.LocalPlayer;
local i,j,k,l=UDim2.new,UDim2.fromOffset,Vector3.new,Color3.fromRGB;
local function m()return h.Character end;
local n,o=task.wait,task.spawn;
local p=Enum.UserInputType.MouseMovement;
local q,r=Enum.Font.GothamBold,Enum.TextXAlignment.Left;
local s,t=Enum.UserInputType.Touch,Enum.UserInputType.MouseButton1;
local function u(v)local w=v.UserInputType;
return w==s or w==t end;
local function x(v)local w=v.UserInputType;
return w==s or w==p end;
if _G.HubBlox and _G.HubBlox.stop then pcall(_G.HubBlox.stop)end;
local y={running=true}_G.HubBlox=y;
pcall(function()if _G.hubAimFly_Close then _G.hubAimFly_Close()end end)local z=400;
local A={delay={0.01,1},forcaTurbo={1,10000},forcaPulo={1,5000},velFly={1,100000},altura={-100000,100000},alcance={0,1000000000},antlim={1,500},flVel={1,100000}}
local B="hub_bloxfruits_v6.json"local C={player=true,npc=true,delay=0.08,turboOn=false,forcaTurbo=100,inf=false,forcaPulo=50,v3=true,v4=true,bot=false,fly=false,alvo="player",velFly=100,altura=0,alcance=0,ant=false,antecipa=2,antlim=30,parado=true,miraOn=true,orbs=true,cadeado=false,antiSit=false,espNpc=false,fl=false,flVel=60,flVis=true,flLock=false,flX=24,flY=-160,botX=20,botY=160,flyX=20,flyY=240}
local D=2.2;
local E=26;
local function F()pcall(function()if writefile then writefile(B,g:JSONEncode(C))end end)end;
pcall(function()if isfile and readfile and isfile(B)then local G=g:JSONDecode(readfile(B))for H,I in pairs(G)do if C[H]~=nil and type(I)==type(C[H])
then C[H]=I end end end end)C.bot,C.fly=false,false;
if C.alvo~="npc"then C.alvo="player"end;
local J={}local function K(L,M)local N=L:Connect(M)table.insert(J,N)return N end;
local O={}local function P(Q,M)O[Q]=O[Q]or{}table.insert(O[Q],M)end;
local function R(Q)for S,M in ipairs(O[Q]or{})do pcall(M)end end;
local function T(U,M)local V,W=pcall(M)if not V then warn("[HubBlox] erro em "..U..": "..tostring(W))end end;
local function X(Y)return Y and Y:FindFirstChildOfClass("Humanoid")end;
local function Z(Y)return Y and(Y:FindFirstChild("HumanoidRootPart")or Y.PrimaryPart)end;
local function _(Y)local a0=X(Y)return a0~=nil and a0.Health>0 end;
local function a1()pcall(function()local a2=Z(m())if a2 then local a3,a4=a2:FindFirstChild("TurboLV"),a2:FindFirstChild("TurboAtt")
if a3 then a3:Destroy()end;
if a4 then a4:Destroy()end end end)end;
a1()local a5;
local a6=Instance.new("ScreenGui")a6.Name="HubBlox"a6.ResetOnSpawn=false;
a6.IgnoreGuiInset=true;
a6.DisplayOrder=999;
local a7=pcall(function()a6.Parent=gethui and gethui()or game:GetService("CoreGui")end)if not a7 or not a6.Parent then a6.Parent=h:WaitForChild("PlayerGui")
end;
y.gui=a6;
local a8=l(8,8,8)local a9=l(22,22,22)local aa=l(235,235,235)local ab=l(55,55,55)local ac=Color3.new(1,1,1)local function ad(N,ae,af)
local v=Instance.new(N)for H,I in pairs(ae)do v[H]=I end;
v.Parent=af;
return v end;
local function ag(ah,a2)ad("UICorner",{CornerRadius=UDim.new(0,a2 or 8)},ah)end;
local ai,aj,ak=300,420,36;
local al=ad("Frame",{Size=j(ai,aj),Position=i(0.5,-ai/2,0.15,0),BackgroundColor3=a8,BackgroundTransparency=.08,Active=true},a6)
ag(al,12)ad("UIStroke",{Color=aa,Thickness=1.5},al)local am=ad("Frame",{Size=i(1,0,0,ak),BackgroundColor3=a9},al)
ag(am,12)ad("TextLabel",{Size=i(1,-90,1,0),Position=i(0,12,0,0),BackgroundTransparency=1,Text="👾 BLOX FRUITS HUB",TextColor3=ac,Font=q,TextSize=15,TextXAlignment=r},am)
local function an(ao,ap,aq)local a4=ad("TextButton",{Size=j(28,24),Position=i(1,ap,0.5,-12),BackgroundColor3=aq,Text=ao,TextColor3=ac,Font=q,TextSize=16},am)
ag(a4,6)return a4 end;
local ar,as=an("-",-68,ab),an("X",-36,ab)local at=ad("ScrollingFrame",{Size=i(1,-12,1,-50),Position=i(0,6,0,44),BackgroundTransparency=1,BorderSizePixel=0,ScrollBarThickness=4,CanvasSize=i(),AutomaticCanvasSize=Enum.AutomaticSize.Y},al)
ad("UIListLayout",{Padding=UDim.new(0,5),SortOrder=Enum.SortOrder.LayoutOrder},at)do local au,av,aw;
K(am.InputBegan,function(v)if u(v)then au,av,aw=true,v.Position,al.Position end end)K(c.InputChanged,function(v)
if au and x(v)then local ax=v.Position-av;
al.Position=i(aw.X.Scale,aw.X.Offset+ax.X,aw.Y.Scale,aw.Y.Offset+ax.Y)end end)K(c.InputEnded,function(v)if u(v)
then au=false end end)end;
local function ay(w)ad("TextLabel",{Size=i(1,-8,0,20),BackgroundTransparency=1,Text="— "..w.." —",TextColor3=aa,Font=q,TextSize=13},at)
end;
local function az(ao,Q)local a4=ad("TextButton",{Size=i(1,-8,0,34),BackgroundColor3=a9,Text="",AutoButtonColor=false},at)
ag(a4)ad("TextLabel",{Size=i(1,-60,1,0),Position=i(0,10,0,0),BackgroundTransparency=1,Text=ao,TextColor3=ac,Font=Enum.Font.Gotham,TextSize=14,TextXAlignment=r},a4)
local aA=ad("Frame",{Size=j(38,18),Position=i(1,-48,.5,-9),BackgroundColor3=ab},a4)ag(aA,9)local aB=ad("Frame",{Size=j(14,14),Position=i(0,2,.5,-7),BackgroundColor3=ac},aA)
ag(aB,7)local function aC()d:Create(aA,TweenInfo.new(.15),{BackgroundColor3=C[Q]and aa or ab}):Play()d:Create(aB,TweenInfo.new(.15),{Position=C[Q]
and i(1,-16,.5,-7)or i(0,2,.5,-7),BackgroundColor3=C[Q]and a8 or ac}):Play()end;
aC()P(Q,aC)K(a4.MouseButton1Click,function()C[Q]=not C[Q]R(Q)F()end)end;
local function aD(ao,Q,aE,aF)local aG,aH=A[Q][1],A[Q][2]local aI=1;
local aJ=ad("Frame",{Size=i(1,-8,0,74),BackgroundColor3=a9},at)ag(aJ)ad("TextLabel",{Size=i(1,-90,0,24),Position=i(0,10,0,2),BackgroundTransparency=1,Text=ao,TextColor3=l(190,190,205),Font=Enum.Font.Gotham,TextSize=12,TextXAlignment=r},aJ)
local aK=ad("TextButton",{Size=j(76,22),Position=i(1,-84,0,4),BackgroundColor3=ab,TextColor3=ac,Font=q,TextSize=11},aJ)
ag(aK,6)local function aL(w,aM)local a4=ad("TextButton",{Size=j(46,34),Position=aM,BackgroundColor3=ab,Text=w,TextColor3=ac,Font=q,TextSize=20},aJ)
ag(a4,6)return a4 end;
local aN=aL("-",i(0,10,0,32))local aO=aL("+",i(1,-56,0,32))local aP=ad("TextBox",{Size=i(1,-124,0,34),Position=i(0,62,0,32),BackgroundColor3=a8,TextColor3=aa,Font=q,TextSize=20,ClearTextOnFocus=false,PlaceholderText="valor",Text=""},aJ)
ag(aP,6)ad("UIStroke",{Color=aa,Thickness=1,Transparency=.5},aP)local function aQ()aP.Text=string.format(aF,C[Q])
aK.Text="passo "..tostring(aE[aI])end;
local function aR(I)I=math.floor(I*10000+0.5)/10000;
C[Q]=math.clamp(I,aG,aH)aQ()F()end;
K(aN.MouseButton1Click,function()aR(C[Q]-aE[aI])end)K(aO.MouseButton1Click,function()aR(C[Q]+aE[aI])end)K(aK.MouseButton1Click,function()
aI=aI%#aE+1;
aQ()end)K(aP.FocusLost,function()local aS=tonumber(aP.Text)if aS then aR(aS)else aQ()end end)aQ()end;
local function aT(ao,Q,aU,aV)local a4=ad("TextButton",{Size=i(1,-8,0,34),BackgroundColor3=a9,Text="",AutoButtonColor=false},at)
ag(a4)local aW=ad("TextLabel",{Size=i(1,-20,1,0),Position=i(0,10,0,0),BackgroundTransparency=1,Text="",TextColor3=ac,Font=q,TextSize=14,TextXAlignment=r},a4)
local function aQ()for v,I in ipairs(aU)do if C[Q]==I then aW.Text=ao..": "..aV[v]end end end;
aQ()P(Q,aQ)K(a4.MouseButton1Click,function()local aX=1;
for v,I in ipairs(aU)do if C[Q]==I then aX=v end end;
C[Q]=aU[aX%#aU+1]R(Q)F()end)end;
T("m",function()ay("COMBATE")az("Bater em Players","player")az("Bater em NPCs","npc")aD("Velocidade do ataque","delay",{0.01,0.05,0.1},"%.2f")
ay("MOVIMENTO")az("Turbo Impulso","turboOn")aD("Velocidade do Turbo","forcaTurbo",{1,10,100},"%g")az("Pulo infinito","inf")
aD("Força do Pulo","forcaPulo",{1,10,100},"%g")ay("RAÇA")az("V3 automático","v3")az("V4 automático","v4")ay("MIRA E VOO")
az("BOT (trava no alvo)","bot")az("FLY (voa até o alvo)","fly")aT("Alvo","alvo",{"player","npc"},{"PLAYER","NPC"})
aD("Velocidade do FLY","velFly",{1,10,100},"%g")aD("Altura do FLY (- = abaixo)","altura",{1,10,100},"%g")aD("Distância (0 = qualquer)","alcance",{1,10,100},"%g")
az("Antecipar movimento","ant")aD("Limite antecipação","antlim",{1,10,100},"%g")az("Parado no ar","parado")az("Bolinha da mira","miraOn")
az("Bolinhas na tela","orbs")az("Cadeado das bolinhas","cadeado")ay("EXTRA")az("Não sentar","antiSit")az("ESP NPC","espNpc")
ay("FLY LIVRE")az("Fly Livre","fl")aD("Velocidade do Fly Livre","flVel",{1,10,100},"%g")az("Mostrar joystick","flVis")
az("Cadeado Fly Livre","flLock")end)local aY=ad("TextButton",{Size=j(46,46),Position=i(0,12,0.5,0),BackgroundColor3=a8,BackgroundTransparency=.05,Text="👾",TextSize=22,Visible=false,AutoButtonColor=false},a6)
ag(aY,23)ad("UIStroke",{Color=ac,Thickness=1.5,Transparency=.3},aY)local aZ=false;
K(ar.MouseButton1Click,function()aZ=not aZ;
at.Visible=not aZ;
al.Size=aZ and j(ai,ak)or j(ai,aj)ar.Text=aZ and"+"or"-"end)K(as.MouseButton1Click,function()al.Visible=false;
aY.Visible=true end)do local au,av,aw,a_=false,nil,nil,0;
K(aY.InputBegan,function(v)if u(v)then au,av,aw,a_=true,v.Position,aY.Position,0;
v.Changed:Connect(function()if v.UserInputState==Enum.UserInputState.End then au=false;
if a_<8 then al.Visible=true;
aY.Visible=false end end end)end end)K(c.InputChanged,function(v)if au and x(v)then local ax=v.Position-av;
a_=math.max(a_,ax.Magnitude)aY.Position=i(aw.X.Scale,aw.X.Offset+ax.X,aw.Y.Scale,aw.Y.Offset+ax.Y)end end)end;
T("b",function()do local b0=ad("Frame",{Name="MiraDot",AnchorPoint=Vector2.new(0.5,0.5),Size=j(4,4),Position=i(0.5,0,0.5,-E),BackgroundColor3=ac,BorderSizePixel=0,Active=false,ZIndex=10,Visible=C.miraOn},a6)
ag(b0,2)ad("UIStroke",{Color=Color3.new(0,0,0),Thickness=1,ApplyStrokeMode=Enum.ApplyStrokeMode.Border},b0)P("miraOn",function()
b0.Visible=C.miraOn end)end;
do local b1=64;
local b2=l(150,150,150)local function b3(ap,b4)local V,b5=pcall(function()return e.CurrentCamera.ViewportSize end)
if V and b5 and b5.X>0 and b5.Y>0 then ap=math.max(0,math.min(ap,b5.X-b1))b4=math.max(0,math.min(b4,b5.Y-b1))end;
return ap,b4 end;
local function b6(ao,Q,b7,b8)local ap,b4=b3(C[b7],C[b8])local ah=ad("TextButton",{Size=j(b1,b1),Position=j(ap,b4),BackgroundColor3=a8,BackgroundTransparency=.35,Text=ao,TextColor3=b2,Font=q,TextSize=16,AutoButtonColor=false,Visible=C.orbs},a6)
ag(ah,b1/2)local b9=ad("UIStroke",{Color=b2,Thickness=1},ah)local function ba()local bb=C[Q]ah.BackgroundTransparency=bb and.05 or.35;
ah.TextColor3=bb and ac or b2;
b9.Color=bb and ac or b2;
b9.Thickness=bb and 3 or 1 end;
ba()P(Q,ba)P("orbs",function()ah.Visible=C.orbs end)local au,av,aw,a_,bc=false,nil,nil,0,nil;
K(ah.InputBegan,function(v)if u(v)then au,av,aw,a_,bc=true,v.Position,ah.Position,0,v end end)K(c.InputChanged,function(v)
if not au then return end;
local bd=v.UserInputType==p or v.UserInputType==s and v==bc;
if not bd then return end;
local ax=v.Position-av;
a_=math.max(a_,ax.Magnitude)if a_>=8 and not C.cadeado then ah.Position=i(aw.X.Scale,aw.X.Offset+ax.X,aw.Y.Scale,aw.Y.Offset+ax.Y)
end end)K(c.InputEnded,function(v)if not au then return end;
local be=v==bc or v.UserInputType==t and bc and bc.UserInputType==t;
if not be then return end;
au=false;
if a_<8 then C[Q]=not C[Q]R(Q)elseif not C.cadeado then C[b7],C[b8]=ah.Position.X.Offset,ah.Position.Y.Offset;
F()end end)end;
b6("BOT","bot","botX","botY")b6("FLY","fly","flyX","flyY")end end)y.stop=function()y.running=false;
pcall(function()b:UnbindFromRenderStep("HubBloxAim")end)for S,N in ipairs(J)do pcall(function()N:Disconnect()end)
end;
pcall(a5)a1()pcall(function()a6:Destroy()end)end;
T("c",function()local bf,bg,bh,bi;
o(function()pcall(function()bf=f:WaitForChild("Remotes",15):WaitForChild("CommF_",15)local bj=f:WaitForChild("Modules",15):WaitForChild("Net",15)
bg=bj:WaitForChild("RE/RegisterAttack",15)bh=bj:WaitForChild("RE/RegisterHit",15)end)pcall(function()bi=f:WaitForChild("Remotes",15):WaitForChild("CommE",15)
end)end)local function bk()local bl,bm,bn={},nil,z;
local bo=Z(m())if not bo then return bl,nil end;
local function bp(bq)local br=Z(bq)if not br or not _(bq)then return end;
local ax=(br.Position-bo.Position).Magnitude;
if ax>z then return end;
table.insert(bl,{bq,bq:FindFirstChild("Head")or br})if ax<=bn then bm,bn=br,ax end end;
if C.npc then local bs=e:FindFirstChild("Enemies")if bs then for S,bt in ipairs(bs:GetChildren())do bp(bt)end end end;
if C.player then for S,ae in ipairs(a:GetPlayers())do if ae~=h and ae.Character then bp(ae.Character)end end end;
return bl,bm end;
o(function()while y.running do local N=m()if bf and N and _(N)and not N:FindFirstChild("HasBuso")then pcall(function()
bf:InvokeServer("Buso")end)end;
n(1)end end)o(function()while y.running do local N=m()local bu=N and N:FindFirstChildOfClass("Tool")if bu and _(N)
and(C.player or C.npc)then local bv,bm=bk()if#bv>0 then if bg then pcall(function()bg:FireServer(0.40000000596046448,1)
end)end;
if bh then pcall(function()bh:FireServer(bv[1][2],bv)end)end;
local bw=bu:FindFirstChild("LeftClickRemote")local bo=Z(N)if bw and bw:IsA("RemoteEvent")and bm and bo then local bx=k(bm.Position.X-bo.Position.X,0,bm.Position.Z-bo.Position.Z)
if bx.Magnitude>0 then pcall(function()bw:FireServer(bx.Unit,1)end)end end end end;
n(math.max(C.delay,A.delay[1]))end end)o(function()n(1)while y.running do local N=m()if C.v3 and bi and N and _(N)
then pcall(function()bi:FireServer("ActivateAbility")end)n(0.5)else n(0.2)end end end)o(function()while y.running do local N=m()
local by=N and N:FindFirstChild("RaceEnergy")local bz=N and N:FindFirstChild("RaceTransformed")if C.v4 and by and bz and by.Value>=1 and not bz.Value and _(N)
then local bA=h:FindFirstChild("Backpack")local bu=bA and bA:FindFirstChild("Awakening")or N:FindFirstChild("Awakening")
local bB=bu and bu:FindFirstChild("RemoteFunction")if bB then pcall(function()bB:InvokeServer(true)end)end end;
n(0.1)end end)end)T("as",function()local function bC(bD)local a0=X(bD)if not a0 then return end;
K(a0.Seated,function(bE)if C.antiSit and bE then pcall(function()a0.Sit=false end)pcall(function()a0:ChangeState(Enum.HumanoidStateType.GettingUp)
end)end end)end;
if m()then bC(m())end;
K(h.CharacterAdded,bC)end)T("esp",function()local bF={}local function bG(v)if bF[v]then return bF[v]end;
local bH=ad("Frame",{BackgroundColor3=l(255,70,70),BorderSizePixel=0,Visible=false},a6)bH.AnchorPoint=Vector2.new(0,0.5)
local ao=ad("TextLabel",{Size=i(0,160,0,32),BackgroundTransparency=1,Text="",TextColor3=ac,Font=Enum.Font.GothamBold,TextSize=13,TextXAlignment=r},a6)
ao.TextStrokeTransparency=0;
ao.Visible=false;
bF[v]={line=bH,txt=ao}return bF[v]end;
local function bI(aS)local v=aS+1;
while bF[v]do bF[v].line.Visible=false;
bF[v].txt.Visible=false;
v=v+1 end end;
K(b.RenderStepped,function()if not C.espNpc then bI(0)return end;
local bJ=e.CurrentCamera;
local bs=e:FindFirstChild("Enemies")if not bJ or not bs then bI(0)return end;
local bo=Z(m())local aS=0;
for S,Y in ipairs(bs:GetChildren())do local a0,a2=X(Y),Z(Y)if a0 and a2 and a0.Health>0 then local aM,bK=bJ:WorldToViewportPoint(a2.Position)
if bK then aS=aS+1;
local bL=bG(aS)local bM,bN=bJ.ViewportSize.X/2,bJ.ViewportSize.Y;
local bO,bP=aM.X-bM,aM.Y-bN;
local bQ=bo and(a2.Position-bo.Position).Magnitude or 0;
bL.line.Position=i(0,bM,0,bN)bL.line.Size=i(0,math.sqrt(bO*bO+bP*bP),0,1)bL.line.Rotation=math.deg(math.atan2(bP,bO))
bL.line.Visible=true;
bL.txt.Position=i(0,aM.X+6,0,aM.Y-10)bL.txt.Text=string.format("%d HP | %dm",math.floor(a0.Health),math.floor(bQ))
bL.txt.Visible=true end end end;
bI(aS)end)end)T("fl",function()local function bR(bS,bT)if CFrame.lookAt then return CFrame.lookAt(bS,bT)end;
return CFrame.new(bS,bT)end;
local bU={}local function bV(bb)if bb then local bD=m()if not bD then return end;
for S,ax in ipairs(bD:GetDescendants())do if ax:IsA("BasePart")then if bU[ax]==nil then bU[ax]=ax.CanCollide end;
ax.CanCollide=false end end else for bW,bX in pairs(bU)do if bW and bW.Parent then bW.CanCollide=bX end;
bU[bW]=nil end end end;
local bY,bZ=64,26;
local b_=ad("Frame",{Size=j(bY,bY),Position=i(0,C.flX,1,C.flY),BackgroundColor3=a8,BackgroundTransparency=.5,Visible=C.flVis and C.fl},a6)
ag(b_,bY/2)ad("UIStroke",{Color=ac,Thickness=1,Transparency=.4},b_)local c0=ad("TextButton",{Size=j(bZ,bZ),Position=i(0.5,-bZ/2,0.5,-bZ/2),BackgroundColor3=ac,Text="",AutoButtonColor=false},b_)
ag(c0,bZ/2)local function c1(ao,b4)local a4=ad("TextButton",{Size=j(44,44),Position=i(0,C.flX+bY+8,1,C.flY+b4),BackgroundColor3=a8,BackgroundTransparency=.35,Text=ao,TextColor3=ac,Font=q,TextSize=18,AutoButtonColor=false,Visible=C.flVis and C.fl},a6)
ag(a4,10)return a4 end;
local c2=c1("▲",-70)local c3=c1("▼",0)P("fl",function()b_.Visible=C.flVis and C.fl;
c2.Visible=b_.Visible;
c3.Visible=b_.Visible end)P("flVis",function()b_.Visible=C.flVis and C.fl;
c2.Visible=b_.Visible;
c3.Visible=b_.Visible end)local c4,c5,c6=0,0,0;
local c7,bc=false,nil;
K(c0.InputBegan,function(v)if u(v)then c7,bc=true,v end end)K(c.InputChanged,function(v)if not c7 or not x(v)then return end;
local c8=b_.AbsolutePosition+b_.AbsoluteSize/2;
local c9=Vector2.new(v.Position.X,v.Position.Y)-c8;
local ca=bY/2;
if c9.Magnitude>ca then c9=c9.Unit*ca end;
c0.Position=i(0.5,c9.X-bZ/2,0.5,c9.Y-bZ/2)c4,c5=c9.X/ca,c9.Y/ca end)K(c.InputEnded,function(v)if c7 and(v==bc or u(v))
then c7=false;
c4,c5=0,0;
c0.Position=i(0.5,-bZ/2,0.5,-bZ/2)end end)local function cb(cc,aC)K(cc.InputBegan,function(v)if u(v)then aC(1)end end)
K(cc.InputEnded,function(v)if u(v)then aC(0)end end)end;
cb(c2,function(I)c6=I end)cb(c3,function(I)c6=-I end)K(b_.InputBegan,function(v)if not u(v)or C.flLock then return end;
local av,aw=v.Position,b_.Position;
local cd;
cd=c.InputChanged:Connect(function(ce)if not x(ce)then return end;
local ax=ce.Position-av;
local cf,cg=aw.X.Offset+ax.X,aw.Y.Offset+ax.Y;
b_.Position=i(0,cf,1,cg)c2.Position=i(0,cf+bY+8,1,cg-70)c3.Position=i(0,cf+bY+8,1,cg)C.flX,C.flY=cf,cg end)local ch;
ch=c.InputEnded:Connect(function(ce)if ce==v or u(ce)then cd:Disconnect()ch:Disconnect()F()end end)end)K(b.RenderStepped,function(ci)
if not C.fl then return end;
local bD=m()local a2,a0=Z(bD),X(bD)if not a2 or not a0 or a0.Health<=0 then return end;
local bJ=e.CurrentCamera;
local cj=bJ and bJ.CFrame.LookVector or k(0,0,-1)local ck=k(cj.X,0,cj.Z)ck=ck.Magnitude>0.01 and ck.Unit or k(0,0,-1)
local cl=k(ck.Z,0,-ck.X)local cm=ck*-c5+cl*c4;
a0.PlatformStand=true;
if cm.Magnitude>0.05 or c6~=0 then if cm.Magnitude>1 then cm=cm.Unit end;
a2.AssemblyLinearVelocity=cm*C.flVel+k(0,c6*C.flVel,0)if cm.Magnitude>0.05 then a2.CFrame=bR(a2.Position,a2.Position+cm)
end else a2.AssemblyLinearVelocity=k(0,0,0)end end)K(b.Stepped,function()if C.fl then pcall(bV,true)end end)P("fl",function()
if not C.fl then local bD=m()local a0=X(bD)if a0 then a0.PlatformStand=false end;
pcall(bV,false)end end)end)T("t",function()local cn=RaycastParams.new()cn.FilterType=Enum.RaycastFilterType.Exclude;
cn.RespectCanCollide=true;
local function co(cp)local cq=cp:FindFirstChild("TurboLV")if cq then return cq end;
local cr=Instance.new("Attachment")cr.Name="TurboAtt"cr.Parent=cp;
cq=Instance.new("LinearVelocity")cq.Name="TurboLV"cq.Attachment0=cr;
cq.RelativeTo=Enum.ActuatorRelativeTo.World;
cq.VelocityConstraintMode=Enum.VelocityConstraintMode.Vector;
cq.ForceLimitMode=Enum.ForceLimitMode.PerAxis;
cq.MaxAxesForce=k(1e9,0,1e9)cq.VectorVelocity=Vector3.zero;
cq.Parent=cp;
return cq end;
local function cs(cp,bx,ct,ci)cn.FilterDescendantsInstances={m()}local cu=math.min(ci,1/20)local cv=ct*cu*2+D;
local cw=ct;
for S,cx in ipairs({-1,0.5,2})do local cy=cp.Position+k(0,cx,0)local a2=e:Raycast(cy,bx*cv,cn)if a2 then local cz=math.max(0,(a2.Distance-D)/(cu*2))
cw=math.min(cw,cz)end end;
return cw end;
local cA=0;
local function cB()if not C.inf then return end;
local cC=os.clock()if cC-cA<0.12 then return end;
cA=cC;
local bD=m()local a0,a2=X(bD),Z(bD)if not a0 or not a2 or a0.Health<=0 then return end;
a0:ChangeState(Enum.HumanoidStateType.Jumping)local I=a2.AssemblyLinearVelocity;
a2.AssemblyLinearVelocity=k(I.X,C.forcaPulo,I.Z)end;
K(c.JumpRequest,cB)K(b.Heartbeat,function(ci)local bD=m()local a0,a2=X(bD),Z(bD)if not a0 or not a2 then return end;
if C.inf and a0.Jump then cB()end;
if not C.turboOn the
