
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
local A={delay={0.01,1},forcaTurbo={1,10000},forcaPulo={1,5000},velFly={1,100000},altura={-100000,100000},alcance={0,1000000000},antlim={1,500},fv={1,100000}}
local B="hub_bloxfruits_v6.json"local C={player=true,npc=true,delay=0.08,turboOn=false,forcaTurbo=100,inf=false,forcaPulo=50,v3=true,v4=true,bot=false,fly=false,npcAlvo=false,velFly=100,altura=0,alcance=0,ant=false,antecipa=2,antlim=30,parado=true,miraOn=true,orbs=true,cadeado=false,asit=false,enp=false,fl=false,fv=60,fw=true,fk=false,fx2=24,flY=-160,botX=20,botY=160,flyX=20,flyY=240}
local D=2.2;
local E=26;
local function F()pcall(function()if writefile then writefile(B,g:JSONEncode(C))end end)end;
pcall(function()if isfile and readfile and isfile(B)then local G=g:JSONDecode(readfile(B))for H,I in pairs(G)do if C[H]~=nil and type(I)==type(C[H])
then C[H]=I end end end end)C.bot,C.fly=false,false;
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
T("m",function()ay("COMBATE")az("Bater em Players","player")az("Bater em NPCs","npc")aD("Velocidade do ataque","delay",{0.01,0.05,0.1},"%.2f")
ay("MOVIMENTO")az("Turbo Impulso","turboOn")aD("Velocidade do Turbo","forcaTurbo",{1,10,100},"%g")az("Pulo infinito","inf")
aD("Força do Pulo","forcaPulo",{1,10,100},"%g")ay("RAÇA")az("V3 automático","v3")az("V4 automático","v4")ay("MIRA E VOO")
az("BOT (trava no alvo)","bot")az("FLY (voa até o alvo)","fly")az("Alvo: NPC (apagado=Player)","npcAlvo")aD("Velocidade do FLY","velFly",{1,10,100},"%g")
aD("Altura do FLY (- = abaixo)","altura",{1,10,100},"%g")aD("Distância (0 = qualquer)","alcance",{1,10,100},"%g")
az("Antecipar movimento","ant")aD("Limite antecipação","antlim",{1,10,100},"%g")az("Parado no ar","parado")az("Bolinha da mira","miraOn")
az("Bolinhas na tela","orbs")az("Cadeado das bolinhas","cadeado")ay("EXTRA")az("Não sentar","asit")az("ESP NPC","enp")
ay("FLY LIVRE")az("Fly Livre","fl")aD("Velocidade do Fly Livre","fv",{1,10,100},"%g")az("Mostrar joystick","fw")
end)local aT=ad("TextButton",{Size=j(46,46),Position=i(0,12,0.5,0),BackgroundColor3=a8,BackgroundTransparency=.05,Text="👾",TextSize=22,Visible=false,AutoButtonColor=false},a6)
ag(aT,23)ad("UIStroke",{Color=ac,Thickness=1.5,Transparency=.3},aT)local aU=false;
K(ar.MouseButton1Click,function()aU=not aU;
at.Visible=not aU;
al.Size=aU and j(ai,ak)or j(ai,aj)ar.Text=aU and"+"or"-"end)K(as.MouseButton1Click,function()al.Visible=false;
aT.Visible=true end)do local au,av,aw,aV=false,nil,nil,0;
K(aT.InputBegan,function(v)if u(v)then au,av,aw,aV=true,v.Position,aT.Position,0;
v.Changed:Connect(function()if v.UserInputState==Enum.UserInputState.End then au=false;
if aV<8 then al.Visible=true;
aT.Visible=false end end end)end end)K(c.InputChanged,function(v)if au and x(v)then local ax=v.Position-av;
aV=math.max(aV,ax.Magnitude)aT.Position=i(aw.X.Scale,aw.X.Offset+ax.X,aw.Y.Scale,aw.Y.Offset+ax.Y)end end)end;
T("b",function()do local aW=ad("Frame",{Name="MiraDot",AnchorPoint=Vector2.new(0.5,0.5),Size=j(4,4),Position=i(0.5,0,0.5,-E),BackgroundColor3=ac,BorderSizePixel=0,Active=false,ZIndex=10,Visible=C.miraOn},a6)
ag(aW,2)ad("UIStroke",{Color=Color3.new(0,0,0),Thickness=1,ApplyStrokeMode=Enum.ApplyStrokeMode.Border},aW)P("miraOn",function()
aW.Visible=C.miraOn end)end;
do local aX=64;
local aY=l(150,150,150)local function aZ(ap,a_)local V,b0=pcall(function()return e.CurrentCamera.ViewportSize end)
if V and b0 and b0.X>0 and b0.Y>0 then ap=math.max(0,math.min(ap,b0.X-aX))a_=math.max(0,math.min(a_,b0.Y-aX))end;
return ap,a_ end;
local function b1(ao,Q,b2,b3)local ap,a_=aZ(C[b2],C[b3])local ah=ad("TextButton",{Size=j(aX,aX),Position=j(ap,a_),BackgroundColor3=a8,BackgroundTransparency=.35,Text=ao,TextColor3=aY,Font=q,TextSize=16,AutoButtonColor=false,Visible=C.orbs},a6)
ag(ah,aX/2)local b4=ad("UIStroke",{Color=aY,Thickness=1},ah)local function b5()local b6=C[Q]ah.BackgroundTransparency=b6 and.05 or.35;
ah.TextColor3=b6 and ac or aY;
b4.Color=b6 and ac or aY;
b4.Thickness=b6 and 3 or 1 end;
b5()P(Q,b5)P("orbs",function()ah.Visible=C.orbs end)local au,av,aw,aV,b7=false,nil,nil,0,nil;
K(ah.InputBegan,function(v)if u(v)then au,av,aw,aV,b7=true,v.Position,ah.Position,0,v end end)K(c.InputChanged,function(v)
if not au then return end;
local b8=v.UserInputType==p or v.UserInputType==s and v==b7;
if not b8 then return end;
local ax=v.Position-av;
aV=math.max(aV,ax.Magnitude)if aV>=8 and not C.cadeado then ah.Position=i(aw.X.Scale,aw.X.Offset+ax.X,aw.Y.Scale,aw.Y.Offset+ax.Y)
end end)K(c.InputEnded,function(v)if not au then return end;
local b9=v==b7 or v.UserInputType==t and b7 and b7.UserInputType==t;
if not b9 then return end;
au=false;
if aV<8 then C[Q]=not C[Q]R(Q)elseif not C.cadeado then C[b2],C[b3]=ah.Position.X.Offset,ah.Position.Y.Offset;
F()end end)end;
b1("BOT","bot","botX","botY")b1("FLY","fly","flyX","flyY")end end)y.stop=function()y.running=false;
pcall(function()b:UnbindFromRenderStep("HubBloxAim")end)for S,N in ipairs(J)do pcall(function()N:Disconnect()end)
end;
pcall(a5)a1()pcall(function()a6:Destroy()end)end;
T("c",function()local ba,bb,bc,bd;
o(function()pcall(function()ba=f:WaitForChild("Remotes",15):WaitForChild("CommF_",15)local be=f:WaitForChild("Modules",15):WaitForChild("Net",15)
bb=be:WaitForChild("RE/RegisterAttack",15)bc=be:WaitForChild("RE/RegisterHit",15)end)pcall(function()bd=f:WaitForChild("Remotes",15):WaitForChild("CommE",15)
end)end)local function bf()local bg,bh,bi={},nil,z;
local bj=Z(m())if not bj then return bg,nil end;
local function bk(bl)local bm=Z(bl)if not bm or not _(bl)then return end;
local ax=(bm.Position-bj.Position).Magnitude;
if ax>z then return end;
table.insert(bg,{bl,bl:FindFirstChild("Head")or bm})if ax<=bi then bh,bi=bm,ax end end;
if C.npc then local bn=e:FindFirstChild("Enemies")if bn then for S,bo in ipairs(bn:GetChildren())do bk(bo)end end end;
if C.player then for S,ae in ipairs(a:GetPlayers())do if ae~=h and ae.Character then bk(ae.Character)end end end;
return bg,bh end;
o(function()while y.running do local N=m()if ba and N and _(N)and not N:FindFirstChild("HasBuso")then pcall(function()
ba:InvokeServer("Buso")end)end;
n(1)end end)o(function()while y.running do local N=m()local bp=N and N:FindFirstChildOfClass("Tool")if bp and _(N)
and(C.player or C.npc)then local bq,bh=bf()if#bq>0 then if bb then pcall(function()bb:FireServer(0.40000000596046448,1)
end)end;
if bc then pcall(function()bc:FireServer(bq[1][2],bq)end)end;
local br=bp:FindFirstChild("LeftClickRemote")local bj=Z(N)if br and br:IsA("RemoteEvent")and bh and bj then local bs=k(bh.Position.X-bj.Position.X,0,bh.Position.Z-bj.Position.Z)
if bs.Magnitude>0 then pcall(function()br:FireServer(bs.Unit,1)end)end end end end;
n(math.max(C.delay,A.delay[1]))end end)o(function()n(1)while y.running do local N=m()if C.v3 and bd and N and _(N)
then pcall(function()bd:FireServer("ActivateAbility")end)n(0.5)else n(0.2)end end end)o(function()while y.running do local N=m()
local bt=N and N:FindFirstChild("RaceEnergy")local bu=N and N:FindFirstChild("RaceTransformed")if C.v4 and bt and bu and bt.Value>=1 and not bu.Value and _(N)
then local bv=h:FindFirstChild("Backpack")local bp=bv and bv:FindFirstChild("Awakening")or N:FindFirstChild("Awakening")
local bw=bp and bp:FindFirstChild("RemoteFunction")if bw then pcall(function()bw:InvokeServer(true)end)end end;
n(0.1)end end)end)T("as",function()local function bx(by)local a0=X(by)if not a0 then return end;
K(a0.Seated,function(bz)if C.asit and bz then pcall(function()a0.Sit=false end)pcall(function()a0:ChangeState(Enum.HumanoidStateType.GettingUp)
end)end end)end;
if m()then bx(m())end;
K(h.CharacterAdded,bx)end)T("esp",function()local bA={}local function bB(v)if bA[v]then return bA[v]end;
local bC=ad("Frame",{BackgroundColor3=l(255,70,70),BorderSizePixel=0,Visible=false},a6)bC.AnchorPoint=Vector2.new(0,0.5)
local ao=ad("TextLabel",{Size=i(0,160,0,32),BackgroundTransparency=1,Text="",TextColor3=ac,Font=Enum.Font.GothamBold,TextSize=13,TextXAlignment=r},a6)
ao.TextStrokeTransparency=0;
ao.Visible=false;
bA[v]={line=bC,txt=ao}return bA[v]end;
local function bD(aS)local v=aS+1;
while bA[v]do bA[v].line.Visible=false;
bA[v].txt.Visible=false;
v=v+1 end end;
K(b.RenderStepped,function()if not C.enp then bD(0)return end;
local bE=e.CurrentCamera;
local bn=e:FindFirstChild("Enemies")if not bE or not bn then bD(0)return end;
local bj=Z(m())local aS=0;
for S,Y in ipairs(bn:GetChildren())do local a0,a2=X(Y),Z(Y)if a0 and a2 and a0.Health>0 then local aM,bF=bE:WorldToViewportPoint(a2.Position)
if bF then aS=aS+1;
local bG=bB(aS)local bH,bI=bE.ViewportSize.X/2,bE.ViewportSize.Y;
local bJ,bK=aM.X-bH,aM.Y-bI;
local bL=bj and(a2.Position-bj.Position).Magnitude or 0;
bG.line.Position=i(0,bH,0,bI)bG.line.Size=i(0,math.sqrt(bJ*bJ+bK*bK),0,1)bG.line.Rotation=math.deg(math.atan2(bK,bJ))
bG.line.Visible=true;
bG.txt.Position=i(0,aM.X+6,0,aM.Y-10)bG.txt.Text=string.format("%d|%dm",math.floor(a0.Health),math.floor(bL))bG.txt.Visible=true end end end;
bD(aS)end)end)T("fl",function()local function bM(bN,bO)if CFrame.lookAt then return CFrame.lookAt(bN,bO)end;
return CFrame.new(bN,bO)end;
local bP={}local function bQ(b6)if b6 then local by=m()if not by then return end;
for S,ax in ipairs(by:GetDescendants())do if ax:IsA("BasePart")then if bP[ax]==nil then bP[ax]=ax.CanCollide end;
ax.CanCollide=false end end else for bR,bS in pairs(bP)do if bR and bR.Parent then bR.CanCollide=bS end;
bP[bR]=nil end end end;
local bT,bU=64,26;
local bV=ad("Frame",{Size=j(bT,bT),Position=i(0,C.fx2,1,C.fy2),BackgroundColor3=a8,BackgroundTransparency=.5,Visible=C.fw and C.fl},a6)
ag(bV,bT/2)ad("UIStroke",{Color=ac,Thickness=1,Transparency=.4},bV)local bW=ad("TextButton",{Size=j(bU,bU),Position=i(0.5,-bU/2,0.5,-bU/2),BackgroundColor3=ac,Text="",AutoButtonColor=false},bV)
ag(bW,bU/2)local function bX(ao,a_)local a4=ad("TextButton",{Size=j(44,44),Position=i(0,C.fx2+bT+8,1,C.fy2+a_),BackgroundColor3=a8,BackgroundTransparency=.35,Text=ao,TextColor3=ac,Font=q,TextSize=18,AutoButtonColor=false,Visible=C.fw and C.fl},a6)
ag(a4,10)return a4 end;
local bY=bX("▲",-70)local bZ=bX("▼",0)local function b_()bV.Visible=C.fw and C.fl;
bY.Visible=bV.Visible;
bZ.Visible=bV.Visible end;
P("fl",b_)P("fw",b_)local c0,c1,c2=0,0,0;
local c3,b7=false,nil;
K(bW.InputBegan,function(v)if u(v)then c3,b7=true,v end end)K(c.InputChanged,function(v)if not c3 or not x(v)then return end;
local c4=bV.AbsolutePosition+bV.AbsoluteSize/2;
local c5=Vector2.new(v.Position.X,v.Position.Y)-c4;
local c6=bT/2;
if c5.Magnitude>c6 then c5=c5.Unit*c6 end;
bW.Position=i(0.5,c5.X-bU/2,0.5,c5.Y-bU/2)c0,c1=c5.X/c6,c5.Y/c6 end)K(c.InputEnded,function(v)if c3 and(v==b7 or u(v))
then c3=false;
c0,c1=0,0;
bW.Position=i(0.5,-bU/2,0.5,-bU/2)end end)local function c7(c8,aC)K(c8.InputBegan,function(v)if u(v)then aC(1)end end)
K(c8.InputEnded,function(v)if u(v)then aC(0)end end)end;
c7(bY,function(I)c2=I end)c7(bZ,function(I)c2=-I end)K(b.RenderStepped,function(c9)if not C.fl then return end;
local by=m()local a2,a0=Z(by),X(by)if not a2 or not a0 or a0.Health<=0 then return end;
local bE=e.CurrentCamera;
local ca=bE and bE.CFrame.LookVector or k(0,0,-1)local cb=k(ca.X,0,ca.Z)cb=cb.Magnitude>0.01 and cb.Unit or k(0,0,-1)
local cc=k(cb.Z,0,-cb.X)local cd=cb*-c1+cc*c0;
a0.PlatformStand=true;
if cd.Magnitude>0.05 or c2~=0 then if cd.Magnitude>1 then cd=cd.Unit end;
a2.AssemblyLinearVelocity=cd*C.fv+k(0,c2*C.fv,0)if cd.Magnitude>0.05 then a2.CFrame=bM(a2.Position,a2.Position+cd)
end else a2.AssemblyLinearVelocity=k(0,0,0)end end)K(b.Stepped,function()if C.fl then pcall(bQ,true)end end)P("fl",function()
if not C.fl then local by=m()local a0=X(by)if a0 then a0.PlatformStand=false end;
pcall(bQ,false)end end)end)T("t",function()local ce=RaycastParams.new()ce.FilterType=Enum.RaycastFilterType.Exclude;
ce.RespectCanCollide=true;
local function cf(cg)local ch=cg:FindFirstChild("TurboLV")if ch then return ch end;
local ci=Instance.new("Attachment")ci.Name="TurboAtt"ci.Parent=cg;
ch=Instance.new("LinearVelocity")ch.Name="TurboLV"ch.Attachment0=ci;
ch.RelativeTo=Enum.ActuatorRelativeTo.World;
ch.VelocityConstraintMode=Enum.VelocityConstraintMode.Vector;
ch.ForceLimitMode=Enum.ForceLimitMode.PerAxis;
ch.MaxAxesForce=k(1e9,0,1e9)ch.VectorVelocity=Vector3.zero;
ch.Parent=cg;
return ch end;
local function cj(cg,bs,ck,c9)ce.FilterDescendantsInstances={m()}local cl=math.min(c9,1/20)local cm=ck*cl*2+D;
local cn=ck;
for S,co in ipairs({-1,0.5,2})do local cp=cg.Position+k(0,co,0)local a2=e:Raycast(cp,bs*cm,ce)if a2 then local cq=math.max(0,(a2.Distance-D)/(cl*2))
cn=math.min(cn,cq)end end;
return cn end;
local cr=0;
local function cs()if not C.inf then return end;
local ct=os.clock()if ct-cr<0.12 then return end;
cr=ct;
local by=m()local a0,a2=X(by),Z(by)if not a0 or not a2 or a0.Health<=0 then return end;
a0:ChangeState(Enum.HumanoidStateType.Jumping)local I=a2.AssemblyLinearVelocity;
a2.AssemblyLinearVelocity=k(I.X,C.forcaPulo,I.Z)end;
K(c.JumpRequest,cs)K(b.Heartbeat,function(c9)local by=m()local a0,a2=X(by),Z(by)if not a0 or not a2 then return end;
if C.inf and a0.Jump then cs()end;
if not C.turboOn then local cu=a2:FindFirstChild("TurboLV")if cu then cu.Enabled=false end else local ch=cf(a2)
ch.Enabled=true;
local Y=a0.MoveDirection;
local cv=k(Y.X,0,Y.Z)if cv.Magnitude<0.1 then ch.VectorVelocity=Vector3.zero else local bs=cv.Unit;
ch.VectorVelocity=bs*cj(a2,bs,C.forcaTurbo,c9)end end end)end)T("v",function()local cw=0.4;
local cx={t=nil,n=0,s=nil,c={},h=nil,p={},q=-100}local function cy(bN,bO)if CFrame.lookAt then return CFrame.lookAt(bN,bO)
end;
return CFrame.new(bN,bO)end;
local function cz(Y)return Y~=nil and Y.Parent~=nil and _(Y)and Z(Y)~=nil end;
local function cA()local ct=os.clock()if ct-cx.q<1 then return cx.p end;
cx.q=ct;
local cB={}local function bk(Y)if Y and Y:IsA("Model")and Y~=m()and a:GetPlayerFromCharacter(Y)==nil and cz(Y)then table.insert(cB,Y)
end end;
local bn=e:FindFirstChild("Enemies")if bn then for S,Y in ipairs(bn:GetChildren())do bk(Y)end end;
cx.p=cB;
return cB end;
local function cC()local cD=Z(m())if not cD then return nil end;
local cE={}if C.npcAlvo then cE=cA()else for S,ae in ipairs(a:GetPlayers())do if ae~=h and ae.Character and cz(ae.Character)
then table.insert(cE,ae.Character)end end end;
local cF,cG;
for S,Y in ipairs(cE)do if cz(Y)then local ax=(Z(Y)
