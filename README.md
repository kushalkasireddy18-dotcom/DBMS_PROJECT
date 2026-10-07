import { useState, type FormEvent, type ReactNode } from 'react';
import { QueryClient, QueryClientProvider, useQueryClient } from '@tanstack/react-query';
import { Link, Route, Switch, useLocation, Router as WouterRouter } from 'wouter';
import {
  useGetDashboard, useListEmployees, useCreateEmployee, useUpdateEmployee, useDeleteEmployee,
  useListDepartments, useCreateDepartment, useUpdateDepartment, useDeleteDepartment,
  useListShifts, useCreateShift, useUpdateShift, useDeleteShift,
  useListAttendance, useCreateAttendance, useUpdateAttendance, useDeleteAttendance,
  useListLeaveRequests, useCreateLeaveRequest, useUpdateLeaveRequest, useDeleteLeaveRequest,
  getGetDashboardQueryKey, getListEmployeesQueryKey, getListDepartmentsQueryKey, getListShiftsQueryKey,
  getListAttendanceQueryKey, getListLeaveRequestsQueryKey,
} from '@workspace/api-client-react';
import type { Employee, Department, Shift, Attendance, LeaveRequest } from '@workspace/api-client-react';
import {
  Activity, ArrowRight, ArrowUpRight, Bell, CalendarDays, Check,
  CheckCircle2, ChevronDown, Clock3, Coffee, FileClock, Filter, LayoutDashboard, MapPin,
  Menu, MoreHorizontal, Plus, Search, ShieldCheck, Sparkles, Users, X, XCircle, Building2,
} from 'lucide-react';
import NotFound from '@/pages/not-found';

const client = new QueryClient();
const navItems = [
  { href: '/dashboard', label: 'Overview', icon: LayoutDashboard },
  { href: '/employees', label: 'Employees', icon: Users },
  { href: '/attendance', label: 'Attendance', icon: Clock3 },
  { href: '/shifts', label: 'Shifts', icon: CalendarDays },
  { href: '/leave-requests', label: 'Leave requests', icon: FileClock },
  { href: '/departments', label: 'Departments', icon: Building2 },
];
const today = new Date().toISOString().slice(0, 10);
type Kind = 'employee' | 'department' | 'shift' | 'attendance' | 'leave';
type RecordItem = Employee | Department | Shift | Attendance | LeaveRequest;
const labels: Record<string, string> = {
  '/employees': 'Employees', '/attendance': 'Attendance', '/shifts': 'Shifts',
  '/leave-requests': 'Leave requests', '/departments': 'Departments',
};

function AppShell() {
  const [location] = useLocation();
  const [mobileOpen, setMobileOpen] = useState(false);
  return <div className="min-h-[100dvh] bg-background">
    <aside className={`fixed inset-y-0 left-0 z-40 flex w-[252px] flex-col border-r border-sidebar-border bg-sidebar transition-transform md:translate-x-0 ${mobileOpen ? 'translate-x-0' : '-translate-x-full'}`}>
      <div className="flex h-[82px] items-center gap-3 border-b border-sidebar-border px-6">
        <div className="grid h-10 w-10 place-items-center rounded-xl bg-primary text-primary-foreground"><ShieldCheck size={21}/></div>
        <div><div className="font-display text-[15px] font-extrabold tracking-tight">CampusFlow</div><div className="mt-0.5 text-[10px] font-semibold uppercase tracking-[.17em] text-muted-foreground">People operations</div></div>
        <button className="ml-auto md:hidden" aria-label="Close navigation" onClick={()=>setMobileOpen(false)}><X size={18}/></button>
      </div>
      <div className="px-4 pt-7">
        <p className="mb-3 px-3 text-[10px] font-bold uppercase tracking-[.16em] text-muted-foreground">Workspace</p>
        <nav className="space-y-1">{navItems.map(({href,label,icon:Icon})=><Link key={href} href={href} onClick={()=>setMobileOpen(false)} className={`flex h-10 items-center gap-3 rounded-lg px-3 text-[13px] font-semibold transition-colors ${location===href || (location==='/'&&href==='/dashboard') ? 'bg-primary text-primary-foreground shadow-sm' : 'text-sidebar-foreground/75 hover:bg-sidebar-accent hover:text-sidebar-accent-foreground'}`}><Icon size={17} strokeWidth={1.8}/>{label}{href==='/leave-requests'&&<span className="ml-auto rounded-full bg-[#d68c50] px-2 py-0.5 font-mono text-[10px] text-white">!</span>}</Link>)}</nav>
      </div>
      <div className="mt-auto p-4">
        <div className="rounded-xl border border-sidebar-border bg-background/65 p-4">
          <div className="mb-3 flex items-center gap-2 text-[11px] font-bold uppercase tracking-wider text-muted-foreground"><span className="h-1.5 w-1.5 rounded-full bg-primary"/>Campus status</div>
          <div className="font-display text-sm font-bold">All systems in rhythm</div>
          <p className="mt-1 text-xs leading-5 text-muted-foreground">Attendance records are up to date.</p>
          <div className="mt-3 flex items-center justify-between border-t border-border pt-3 text-[10px] text-muted-foreground"><span>LAST SYNC</span><span className="font-mono">just now</span></div>
        </div>
        <div className="mt-4 flex items-center gap-3 px-2 py-2">
          <div className="grid h-9 w-9 place-items-center rounded-full bg-[#ead7bd] font-display text-xs font-bold text-[#754d2e]">HR</div>
          <div className="min-w-0 flex-1"><div className="truncate text-xs font-bold">People team</div><div className="text-[10px] text-muted-foreground">Campus administration</div></div>
          <MoreHorizontal size={17} className="text-muted-foreground"/>
        </div>
      </div>
    </aside>
    {mobileOpen&&<button aria-label="Close menu backdrop" className="fixed inset-0 z-30 bg-foreground/20 md:hidden" onClick={()=>setMobileOpen(false)}/>}
    <main className="min-h-[100dvh] md:pl-[252px]">
      <header className="sticky top-0 z-20 flex h-[68px] items-center border-b border-border/80 bg-background/90 px-5 backdrop-blur-md md:px-9">
        <button className="mr-3 md:hidden" aria-label="Open navigation" onClick={()=>setMobileOpen(true)}><Menu size={19}/></button>
        <div className="flex items-center gap-2 text-xs text-muted-foreground"><span>Workspace</span><span>/</span><span className="font-semibold text-foreground">{location==='/dashboard'||location==='/'?'Overview':labels[location]||'Workspace'}</span></div>
        <div className="ml-auto flex items-center gap-3"><div className="hidden text-right sm:block"><p className="text-[11px] font-semibold">{new Date().toLocaleDateString('en-US',{weekday:'long',month:'long',day:'numeric'})}</p><p className="text-[10px] text-muted-foreground">Academic year {academicYear()}</p></div><button className="relative grid h-9 w-9 place-items-center rounded-lg border border-border bg-card text-muted-foreground hover:text-foreground" aria-label="Notifications"><Bell size={17}/><i className="absolute right-2 top-2 h-1.5 w-1.5 rounded-full bg-[#d68c50]"/></button></div>
      </header>
      <div className="mx-auto max-w-[1480px] px-5 py-7 md:px-9 md:py-9">
        <Switch>
          <Route path="/" component={DashboardPage}/>
          <Route path="/dashboard" component={DashboardPage}/>
          <Route path="/employees"><RecordsPage kind="employee"/></Route>
          <Route path="/attendance"><RecordsPage kind="attendance"/></Route>
          <Route path="/shifts"><RecordsPage kind="shift"/></Route>
          <Route path="/leave-requests"><RecordsPage kind="leave"/></Route>
          <Route path="/departments"><RecordsPage kind="department"/></Route>
          <Route component={NotFound}/>
        </Switch>
      </div>
    </main>
  </div>;
}

function DashboardPage() {
  const {data:dash,isLoading,isError,refetch}=useGetDashboard();
  const {data:requests=[]}=useListLeaveRequests({status:'Pending'});
  const {data:activityEmployees=[]}=useListEmployees();
  const [,navigate]=useLocation();
  if(isLoading)return <SkeletonPage/>;
  if(isError||!dash)return <ErrorState retry={()=>void refetch()}/>;
  const presentPct=dash.totalEmployees?Math.round(dash.presentToday/dash.totalEmployees*100):0;
  const maxDay=Math.max(...(dash.attendanceByDay||[]).map(d=>d.present+d.absent),1);
  return <div className="enter space-y-7">
    <div className="flex flex-col justify-between gap-4 sm:flex-row sm:items-end">
      <div><div className="mb-2 flex items-center gap-2 text-[11px] font-bold uppercase tracking-[.16em] text-primary"><Sparkles size={13}/> {new Date().toLocaleDateString('en-US',{weekday:'long',month:'long',day:'numeric',year:'numeric'})}</div><h1 className="font-display text-[30px] font-bold tracking-[-.045em] md:text-[36px]">Good morning, team.</h1><p className="mt-1.5 text-sm text-muted-foreground">A clear view of the people and rhythms across campus.</p></div>
      <button onClick={()=>navigate('/attendance')} className="inline-flex h-10 items-center justify-center gap-2 rounded-lg bg-primary px-4 text-sm font-semibold text-primary-foreground shadow-sm transition hover:opacity-90"><Plus size={16}/> Record attendance</button>
    </div>
    <section className="grid gap-4 sm:grid-cols-2 xl:grid-cols-4">
      <Metric title="Total employees" value={dash.totalEmployees} sub="Across all departments" icon={<Users size={18}/>} color="green"/>
      <Metric title="Present today" value={dash.presentToday} sub={`${presentPct}% of your team`} icon={<CheckCircle2 size={18}/>} color="mint"/>
      <Metric title="On leave" value={dash.onLeave} sub="Out of office today" icon={<Coffee size={18}/>} color="sand"/>
      <Metric title="Needs review" value={dash.pendingRequests} sub="Leave requests waiting" icon={<FileClock size={18}/>} color="rose" trend={dash.pendingRequests>0}/>
    </section>
    <section className="grid gap-5 xl:grid-cols-[1.55fr_1fr]">
      <div className="panel-shadow rounded-xl border border-border bg-card p-5 md:p-6">
        <div className="mb-6 flex items-start justify-between"><div><h2 className="font-display text-base font-bold">Attendance rhythm</h2><p className="mt-1 text-xs text-muted-foreground">Present and absent, this week</p></div><div className="rounded-lg bg-secondary px-3 py-1.5 text-[11px] font-semibold text-secondary-foreground">This week <ChevronDown className="ml-1 inline" size={13}/></div></div>
        <div className="flex h-[184px] items-end justify-between gap-3 border-b border-border/80 pb-0">
          {(dash.attendanceByDay||[]).map((d,i)=>{const ht=(d.present+d.absent)/maxDay*145;const present=d.present/(d.present+d.absent||1)*100;return <div key={d.day} className="flex h-full flex-1 flex-col items-center justify-end gap-2"><div className="flex w-full max-w-[44px] flex-col justify-end overflow-hidden rounded-t-md bg-[#f0e5d4]" style={{height:`${Math.max(ht,8)}px`}}><div className="w-full rounded-t-sm bg-primary transition-all" style={{height:`${present}%`}}/></div><span className="pb-2 text-[10px] font-medium text-muted-foreground">{d.day}</span></div>})}
        </div><div className="mt-4 flex items-center gap-5 text-[11px] text-muted-foreground"><span className="flex items-center gap-2"><i className="h-2 w-2 rounded-sm bg-primary"/>Present</span><span className="flex items-center gap-2"><i className="h-2 w-2 rounded-sm bg-[#f0e5d4]"/>Absent</span><span className="ml-auto font-mono">{dash.attendanceRate}% avg. attendance</span></div>
      </div>
      <div className="panel-shadow rounded-xl border border-border bg-card p-5 md:p-6">
        <div className="mb-5 flex items-start justify-between"><div><h2 className="font-display text-base font-bold">By department</h2><p className="mt-1 text-xs text-muted-foreground">People across campus</p></div><button aria-label="View departments" onClick={()=>navigate('/departments')} className="grid h-8 w-8 place-items-center rounded-lg border border-border text-muted-foreground hover:bg-muted"><ArrowRight size={15}/></button></div>
        <div className="space-y-[18px]">{(dash.departmentBreakdown||[]).slice(0,5).map((dep,i)=>{const colors=['bg-primary','bg-[#c98c55]','bg-[#789a9b]','bg-[#be7880]','bg-[#8a9e65]'];return <div key={dep.name}><div className="mb-2 flex items-center justify-between text-xs"><span className="font-semibold">{dep.name}</span><span className="font-mono text-muted-foreground">{dep.count}</span></div><div className="h-1.5 overflow-hidden rounded-full bg-muted"><div className={`h-full rounded-full ${colors[i%colors.length]}`} style={{width:`${dash.totalEmployees?dep.count/dash.totalEmployees*100:0}%`}}/></div></div>})}</div>
      </div>
    </section>
    <section className="grid gap-5 xl:grid-cols-[1.2fr_.8fr]">
      <div className="panel-shadow rounded-xl border border-border bg-card">
        <div className="flex items-center justify-between border-b border-border px-5 py-4 md:px-6"><div><h2 className="font-display text-base font-bold">Leave requests</h2><p className="mt-1 text-xs text-muted-foreground">A few things need your attention</p></div><button onClick={()=>navigate('/leave-requests')} className="text-xs font-semibold text-primary hover:underline">See all <ArrowRight className="ml-1 inline" size={13}/></button></div>
        {requests.length?requests.slice(0,3).map(r=><div key={r.id} className="flex items-center gap-3 border-b border-border/70 px-5 py-3.5 last:border-0 md:px-6"><Avatar name={r.employeeName}/><div className="min-w-0 flex-1"><p className="truncate text-xs font-semibold">{r.employeeName}</p><p className="mt-1 text-[10px] text-muted-foreground">{r.type} · {r.days} days · {r.departmentName}</p></div><button onClick={()=>navigate('/leave-requests')} className="rounded-md border border-border px-2.5 py-1.5 text-[10px] font-semibold hover:bg-secondary">Review</button></div>):<EmptyInline label="No pending leave requests"/>}
      </div>
      <div className="panel-shadow rounded-xl border border-border bg-card">
        <div className="flex items-center justify-between border-b border-border px-5 py-4 md:px-6"><div><h2 className="font-display text-base font-bold">Recent activity</h2><p className="mt-1 text-xs text-muted-foreground">The latest updates from your team</p></div><Activity size={17} className="text-muted-foreground"/></div>
        {dash.recentActivity?.length?dash.recentActivity.slice(0,4).map((a,i)=><div key={`${a.id}-${i}`} className="flex items-start gap-3 border-b border-border/70 px-5 py-3 last:border-0 md:px-6"><div className="mt-0.5 grid h-7 w-7 shrink-0 place-items-center rounded-lg bg-secondary text-primary"><Activity size={14}/></div><div className="min-w-0 flex-1"><p className="text-xs font-semibold">{a.title}</p><p className="mt-1 truncate text-[10px] text-muted-foreground">{a.detail}</p></div><span className="whitespace-nowrap font-mono text-[9px] text-muted-foreground">{relative(a.date)}</span></div>):activityEmployees.slice(0,3).map(e=><div key={e.id} className="flex items-center gap-3 border-b border-border/70 px-5 py-3 last:border-0"><Avatar name={`${e.firstName} ${e.lastName}`}/><div className="min-w-0"><p className="text-xs font-semibold">{e.firstName} {e.lastName}</p><p className="text-[10px] text-muted-foreground">Part of {e.departmentName}</p></div><span className="ml-auto font-mono text-[9px] text-muted-foreground">{relative(e.joinedAt)}</span></div>)}
      </div>
    </section>
  </div>;
}

function Metric({title,value,sub,icon,color,trend}:{title:string;value:number|string;sub:string;icon:ReactNode;color:string;trend?:boolean}) {
  const colorMap:Record<string,string>={green:'bg-[#e3ede5] text-[#35644c]',mint:'bg-[#e0efea] text-[#367565]',sand:'bg-[#f3e8d6] text-[#a06a37]',rose:'bg-[#f3e4e3] text-[#a9524e]'};
  return <div className="panel-shadow rounded-xl border border-border bg-card p-5"><div className="flex items-center justify-between"><span className="text-xs font-semibold text-muted-foreground">{title}</span><span className={`grid h-9 w-9 place-items-center rounded-lg ${colorMap[color]}`}>{icon}</span></div><div className="mt-4 flex items-end justify-between"><span className="font-display text-[32px] font-bold tracking-[-.04em] leading-none">{value}</span>{trend&&<span className="flex items-center gap-1 rounded-full bg-[#fbefdf] px-2 py-1 text-[10px] font-semibold text-[#97622c]"><ArrowUpRight size={12}/>Action</span>}</div><p className="mt-2 text-[11px] text-muted-foreground">{sub}</p></div>;
}

function RecordsPage({kind}:{kind:Kind}) {
  const [search,setSearch]=useState(''); const [status,setStatus]=useState(''); const [department,setDepartment]=useState(''); const [day,setDay]=useState('');
  const [modal,setModal]=useState<{mode:'create'|'edit'|'view';item?:RecordItem}|null>(null);
  const [error,setError]=useState('');
  const {data:employees=[],isLoading:empLoading,isError:empError,refetch:refetchEmp}=useListEmployees({search:kind==='employee'?search:undefined,departmentId:department?Number(department):undefined,status:status||undefined});
  const {data:departments=[],isLoading:depLoading,isError:depError,refetch:refetchDep}=useListDepartments({search:kind==='department'?search:undefined});
  const {data:shifts=[],isLoading:shiftLoading,isError:shiftError,refetch:refetchShift}=useListShifts({search:kind==='shift'?search:undefined});
  const {data:attendance=[],isLoading:attLoading,isError:attError,refetch:refetchAtt}=useListAttendance({search:kind==='attendance'?search:undefined,departmentId:department?Number(department):undefined,status:status||undefined,date:day||undefined});
  const {data:leaves=[],isLoading:leaveLoading,isError:leaveError,refetch:refetchLeave}=useListLeaveRequests({search:kind==='leave'?search:undefined,departmentId:department?Number(department):undefined,status:status||undefined});
  const qc=useQueryClient();
  const ce=useCreateEmployee(),ue=useUpdateEmployee(),de=useDeleteEmployee();
  const cd=useCreateDepartment(),ud=useUpdateDepartment(),dd=useDeleteDepartment();
  const cs=useCreateShift(),us=useUpdateShift(),ds=useDeleteShift();
  const ca=useCreateAttendance(),ua=useUpdateAttendance(),da=useDeleteAttendance();
  const cl=useCreateLeaveRequest(),ul=useUpdateLeaveRequest(),dl=useDeleteLeaveRequest();
  const configs:Record<Kind,{key:string;title:string;plural:string;icon:ReactNode;loading:boolean;failed:boolean;retry:()=>void;data:RecordItem[]}>={
    employee:{key:'employees',title:'Employee records',plural:'employees',icon:<Users size={17}/>,loading:empLoading,failed:empError,retry:()=>void refetchEmp(),data:employees},
    department:{key:'departments',title:'Departments',plural:'departments',icon:<Building2 size={17}/>,loading:depLoading,failed:depError,retry:()=>void refetchDep(),data:departments},
    shift:{key:'shifts',title:'Shift definitions',plural:'shifts',icon:<CalendarDays size={17}/>,loading:shiftLoading,failed:shiftError,retry:()=>void refetchShift(),data:shifts},
    attendance:{key:'attendance',title:'Daily attendance',plural:'records',icon:<Clock3 size={17}/>,loading:attLoading,failed:attError,retry:()=>void refetchAtt(),data:attendance},
    leave:{key:'leave-requests',title:'Leave requests',plural:'requests',icon:<FileClock size={17}/>,loading:leaveLoading,failed:leaveError,retry:()=>void refetchLeave(),data:leaves},
  };
  const cfg=configs[kind];
  const invalidate=()=>{void qc.invalidateQueries({queryKey:getGetDashboardQueryKey()});void qc.invalidateQueries({queryKey:getListEmployeesQueryKey()});void qc.invalidateQueries({queryKey:getListDepartmentsQueryKey()});void qc.invalidateQueries({queryKey:getListShiftsQueryKey()});void qc.invalidateQueries({queryKey:getListAttendanceQueryKey()});void qc.invalidateQueries({queryKey:getListLeaveRequestsQueryKey()});};
  const itemPayload=(item:RecordItem)=>{
    if(kind==='employee'){const e=item as Employee;return {firstName:e.firstName,lastName:e.lastName,email:e.email,phone:e.phone,departmentId:e.departmentId,role:e.role,shiftId:e.shiftId,status:e.status,joinedAt:e.joinedAt};}
    if(kind==='department'){const d=item as Department;return {name:d.name,manager:d.manager,location:d.location};}
    if(kind==='shift'){const s=item as Shift;return {name:s.name,startTime:s.startTime,endTime:s.endTime,days:s.days};}
    if(kind==='attendance'){const a=item as Attendance;return {employeeId:a.employeeId,date:a.date,checkIn:a.checkIn||'',checkOut:a.checkOut||'',status:a.status};}
    const l=item as LeaveRequest;return {employeeId:l.employeeId,type:l.type,startDate:l.startDate,endDate:l.endDate,days:l.days,reason:l.reason,status:l.status,appliedAt:l.appliedAt};
  };
  const save=async(data:Record<string,any>)=>{
    setError('');
    try {
      if(kind==='employee'){if(modal?.mode==='edit'&&modal.item)await ue.mutateAsync({id:modal.item.id,data:data as any});else await ce.mutateAsync({data:data as any});}
      if(kind==='department'){if(modal?.mode==='edit'&&modal.item)await ud.mutateAsync({id:modal.item.id,data:data as any});else await cd.mutateAsync({data:data as any});}
      if(kind==='shift'){if(modal?.mode==='edit'&&modal.item)await us.mutateAsync({id:modal.item.id,data:data as any});else await cs.mutateAsync({data:data as any});}
      if(kind==='attendance'){if(modal?.mode==='edit'&&modal.item)await ua.mutateAsync({id:modal.item.id,data:data as any});else await ca.mutateAsync({data:data as any});}
      if(kind==='leave'){if(modal?.mode==='edit'&&modal.item)await ul.mutateAsync({id:modal.item.id,data:data as any});else await cl.mutateAsync({data:data as any});}
      invalidate();setModal(null);
    }catch(e){setError(e instanceof Error?e.message:'Unable to save this record.');throw e;}
  };
  const remove=async(item:RecordItem)=>{
    if(!window.confirm(`Delete this ${kind==='leave'?'leave request':kind}? This cannot be undone.`))return;
    setError('');
    try{if(kind==='employee')await de.mutateAsync({id:item.id});if(kind==='department')await dd.mutateAsync({id:item.id});if(kind==='shift')await ds.mutateAsync({id:item.id});if(kind==='attendance')await da.mutateAsync({id:item.id});if(kind==='leave')await dl.mutateAsync({id:item.id});invalidate();}
    catch(e){setError(e instanceof Error?e.message:'Unable to delete this record.');}
  };
  const leaveStatus=async(item:LeaveRequest,next:'Approved'|'Rejected')=>{try{await ul.mutateAsync({id:item.id,data:{...itemPayload(item),status:next} as any});invalidate();}catch(e){setError(e instanceof Error?e.message:'Could not update request.');}};
  return <div className="enter space-y-6">
    <div className="flex flex-col justify-between gap-4 sm:flex-row sm:items-end"><div><div className="mb-2 flex items-center gap-2 text-[11px] font-bold uppercase tracking-[.15em] text-primary">{cfg.icon} People operations</div><h1 className="font-display text-[30px] font-bold tracking-[-.045em]">{cfg.title}</h1><p className="mt-1.5 text-sm text-muted-foreground">{description(kind)}</p></div><button onClick={()=>setModal({mode:'create'})} className="inline-flex h-10 items-center justify-center gap-2 rounded-lg bg-primary px-4 text-sm font-semibold text-primary-foreground shadow-sm hover:opacity-90"><Plus size={16}/> Add {kindLabel(kind)}</button></div>
    <div className="panel-shadow overflow-hidden rounded-xl border border-border bg-card">
      <div className="flex flex-col gap-3 border-b border-border p-4 md:flex-row md:items-center md:px-5">
        <div className="relative flex-1"><Search className="absolute left-3 top-1/2 -translate-y-1/2 text-muted-foreground" size={15}/><input aria-label={`Search ${cfg.plural}`} data-testid={`input-search-${kind}`} value={search} onChange={e=>setSearch(e.target.value)} placeholder={`Search ${cfg.plural}...`} className="h-10 w-full rounded-lg border border-input bg-background pl-9 pr-3 text-xs outline-none placeholder:text-muted-foreground focus:ring-2 focus:ring-ring/30"/></div>
        {kind!=='department'&&kind!=='shift'&&<><div className="relative"><Filter className="pointer-events-none absolute left-3 top-1/2 -translate-y-1/2 text-muted-foreground" size={13}/><select aria-label="Filter by status" value={status} onChange={e=>setStatus(e.target.value)} className="h-10 min-w-[142px] appearance-none rounded-lg border border-input bg-background pl-9 pr-8 text-xs outline-none focus:ring-2 focus:ring-ring/30"><option value="">All statuses</option>{statusOptions(kind).map(s=><option key={s}>{s}</option>)}</select><ChevronDown className="pointer-events-none absolute right-3 top-1/2 -translate-y-1/2 text-muted-foreground" size={13}/></div></>}
        {kind==='attendance'&&<input aria-label="Filter attendance by date" type="date" value={day} onChange={e=>setDay(e.target.value)} className="h-10 rounded-lg border border-input bg-background px-3 text-xs outline-none focus:ring-2 focus:ring-ring/30"/>}
        {(kind==='employee'||kind==='attendance'||kind==='leave')&&<div className="relative"><select aria-label="Filter by department" value={department} onChange={e=>setDepartment(e.target.value)} className="h-10 min-w-[156px] appearance-none rounded-lg border border-input bg-background px-3 pr-8 text-xs outline-none focus:ring-2 focus:ring-ring/30"><option value="">All departments</option>{departments.map(d=><option key={d.id} value={d.id}>{d.name}</option>)}</select><ChevronDown className="pointer-events-none absolute right-3 top-1/2 -translate-y-1/2 text-muted-foreground" size={13}/></div>}
        <div className="flex h-10 items-center rounded-lg bg-muted px-3 text-[11px] font-semibold text-muted-foreground">{cfg.data.length} {cfg.data.length===1?kindLabel(kind):cfg.plural}</div>
      </div>
      {error&&<div role="alert" className="mx-5 mt-4 flex items-center justify-between rounded-lg border border-destructive/20 bg-destructive/5 px-3 py-2 text-xs text-destructive"><span>{error}</span><button onClick={()=>setError('')} aria-label="Dismiss error"><X size={14}/></button></div>}
      {cfg.loading?<TableSkeleton/>:cfg.failed?<div className="p-7"><ErrorState retry={cfg.retry}/></div>:cfg.data.length===0?<EmptyState kind={kind} onAdd={()=>setModal({mode:'create'})}/>:<RecordsTable kind={kind} rows={cfg.data} onView={item=>setModal({mode:'view',item})} onEdit={item=>setModal({mode:'edit',item})} onDelete={remove} onLeaveStatus={leaveStatus}/>}
      <div className="flex flex-col gap-2 border-t border-border px-5 py-3 text-[10px] text-muted-foreground sm:flex-row sm:items-center sm:justify-between"><span>Showing {cfg.data.length} {cfg.data.length===1?'record':cfg.plural}</span><span className="font-mono">Live campus records</span></div>
    </div>
    {modal&&<RecordModal kind={kind} mode={modal.mode} item={modal.item} departments={departments} employees={employees} shifts={shifts} onClose={()=>{setModal(null);setError('')}} onSave={save}/>}
  </div>;
}

function RecordsTable({kind,rows,onView,onEdit,onDelete,onLeaveStatus}:{kind:Kind;rows:RecordItem[];onView:(r:RecordItem)=>void;onEdit:(r:RecordItem)=>void;onDelete:(r:RecordItem)=>void;onLeaveStatus:(r:LeaveRequest,s:'Approved'|'Rejected')=>void}) {
  const columns:Record<Kind,string[]>={employee:['Employee','Department','Role','Shift','Status',''],department:['Department','Manager','Location','Employees',''],shift:['Shift','Hours','Days','Employees',''],attendance:['Employee','Department','Date','Check in','Check out','Status',''],leave:['Employee','Leave type','Dates','Days','Reason','Status','']};
  return <div className="overflow-x-auto"><table className="w-full min-w-[760px] border-collapse text-left"><thead><tr className="bg-muted/55">{columns[kind].map((col,i)=><th key={`${col}-${i}`} className="px-5 py-3 text-[9px] font-bold uppercase tracking-[.13em] text-muted-foreground">{col}</th>)}</tr></thead><tbody>{rows.map((row,index)=><tr key={row.id} className="group border-t border-border/70 transition-colors hover:bg-secondary/30" data-testid={`row-${kind}-${row.id}`}><RecordCells kind={kind} row={row} onClick={()=>onView(row)}/><td className="px-4 py-3"><div className="flex items-center justify-end gap-1.5">
    {kind==='leave'&&(row as LeaveRequest).status==='Pending'&&<><button onClick={()=>onLeaveStatus(row as LeaveRequest,'Approved')} title="Approve request" className="grid h-8 w-8 place-items-center rounded-md text-primary hover:bg-secondary"><Check size={15}/></button><button onClick={()=>onLeaveStatus(row as LeaveRequest,'Rejected')} title="Reject request" className="grid h-8 w-8 place-items-center rounded-md text-destructive hover:bg-destructive/10"><X size={15}/></button></>}
    <button onClick={()=>onEdit(row)} title="Edit record" className="rounded-md px-2 py-1.5 text-[10px] font-semibold text-muted-foreground hover:bg-muted hover:text-foreground">Edit</button><button onClick={()=>onDelete(row)} title="Delete record" className="grid h-8 w-8 place-items-center rounded-md text-muted-foreground hover:bg-destructive/10 hover:text-destructive"><XCircle size={15}/></button>
  </div></td></tr>)}</tbody></table></div>;
}
function RecordCells({kind,row,onClick}:{kind:Kind;row:RecordItem;onClick:()=>void}) {
  if(kind==='employee'){const e=row as Employee;return <><td className="px-5 py-3"><button onClick={onClick} className="flex items-center gap-3 text-left"><Avatar name={`${e.firstName} ${e.lastName}`}/><div><p className="text-xs font-semibold">{e.firstName} {e.lastName}</p><p className="mt-0.5 text-[10px] text-muted-foreground">{e.email}</p></div></button></td><td className="px-5 py-3 text-xs">{e.departmentName}</td><td className="px-5 py-3 text-xs text-muted-foreground">{e.role}</td><td className="px-5 py-3 text-xs text-muted-foreground">{e.shiftName}</td><td className="px-5 py-3"><StatusBadge value={e.status}/></td></>}
  if(kind==='department'){const d=row as Department;return <><td className="px-5 py-3"><button onClick={onClick} className="flex items-center gap-3 text-left"><span className="grid h-9 w-9 place-items-center rounded-lg bg-secondary text-primary"><Building2 size={16}/></span><span className="text-xs font-semibold">{d.name}</span></button></td><td className="px-5 py-3 text-xs">{d.manager}</td><td className="px-5 py-3 text-xs text-muted-foreground"><MapPin size={12} className="mr-1 inline"/>{d.location}</td><td className="px-5 py-3 font-mono text-xs">{d.employeeCount}</td></>}
  if(kind==='shift'){const s=row as Shift;return <><td className="px-5 py-3"><button onClick={onClick} className="flex items-center gap-3 text-left"><span className="grid h-9 w-9 place-items-center rounded-lg bg-[#f3e8d6] text-[#a06a37]"><Clock3 size={16}/></span><span className="text-xs font-semibold">{s.name}</span></button></td><td className="px-5 py-3 font-mono text-[11px]">{s.startTime} – {s.endTime}</td><td className="px-5 py-3 text-xs text-muted-foreground">{s.days}</td><td className="px-5 py-3 font-mono text-xs">{s.employeeCount}</td></>}
  if(kind==='attendance'){const a=row as Attendance;return <><td className="px-5 py-3"><button onClick={onClick} className="flex items-center gap-3 text-left"><Avatar name={a.employeeName}/><span className="text-xs font-semibold">{a.employeeName}</span></button></td><td className="px-5 py-3 text-xs">{a.departmentName}</td><td className="px-5 py-3 font-mono text-[10px]">{fmtDate(a.date)}</td><td className="px-5 py-3 font-mono text-[10px]">{a.checkIn||'—'}</td><td className="px-5 py-3 font-mono text-[10px]">{a.checkOut||'—'}</td><td className="px-5 py-3"><StatusBadge value={a.status}/></td></>}
  const l=row as LeaveRequest;return <><td className="px-5 py-3"><button onClick={onClick} className="flex items-center gap-3 text-left"><Avatar name={l.employeeName}/><span><b className="block text-xs font-semibold">{l.employeeName}</b><span className="text-[10px] text-muted-foreground">{l.departmentName}</span></span></button></td><td className="px-5 py-3 text-xs">{l.type}</td><td className="px-5 py-3 text-[10px] text-muted-foreground">{fmtDate(l.startDate)} – {fmtDate(l.endDate)}</td><td className="px-5 py-3 font-mono text-xs">{l.days}</td><td className="max-w-[220px] truncate px-5 py-3 text-[10px] text-muted-foreground">{l.reason}</td><td className="px-5 py-3"><StatusBadge value={l.status}/></td></>;
}

function RecordModal({kind,mode,item,departments,employees,shifts,onClose,onSave}:{kind:Kind;mode:'create'|'edit'|'view';item?:RecordItem;departments:Department[];employees:Employee[];shifts:Shift[];onClose:()=>void;onSave:(data:Record<string,any>)=>void}) {
  const initial=item?payloadFor(kind,item):defaults(kind);
  const [values,setValues]=useState<Record<string,any>>(initial);
  const [busy,setBusy]=useState(false);
  const [error,setError]=useState('');
  const readOnly=mode==='view';
  const fields:Record<Kind,{key:string;label:string;type?:string;placeholder?:string;options?:{value:string;label:string}[]}[]> = {
    employee:[{key:'firstName',label:'First name'},{key:'lastName',label:'Last name'},{key:'email',label:'Email',type:'email'},{key:'phone',label:'Phone',type:'tel'},{key:'departmentId',label:'Department',type:'select',options:departments.map(d=>({value:String(d.id),label:d.name}))},{key:'role',label:'Role'},{key:'shiftId',label:'Shift',type:'select',options:shifts.map(s=>({value:String(s.id),label:s.name}))},{key:'status',label:'Status',type:'select',options:['Active','On leave','Inactive'].map(x=>({value:x,label:x}))},{key:'joinedAt',label:'Joined on',type:'date'}],
    department:[{key:'name',label:'Department name'},{key:'manager',label:'Department manager'},{key:'location',label:'Campus location'}],
    shift:[{key:'name',label:'Shift name'},{key:'startTime',label:'Start time',type:'time'},{key:'endTime',label:'End time',type:'time'},{key:'days',label:'Working days',placeholder:'Monday–Friday'}],
    attendance:[{key:'employeeId',label:'Employee',type:'select',options:employees.map(e=>({value:String(e.id),label:`${e.firstName} ${e.lastName}`}))},{key:'date',label:'Date',type:'date'},{key:'checkIn',label:'Check in',type:'time'},{key:'checkOut',label:'Check out',type:'time'},{key:'status',label:'Status',type:'select',options:['Present','Late','Absent','Half day'].map(x=>({value:x,label:x}))}],
    leave:[{key:'employeeId',label:'Employee',type:'select',options:employees.map(e=>({value:String(e.id),label:`${e.firstName} ${e.lastName}`}))},{key:'type',label:'Leave type',type:'select',options:['Annual','Sick','Personal','Unpaid'].map(x=>({value:x,label:x}))},{key:'startDate',label:'First day',type:'date'},{key:'endDate',label:'Last day',type:'date'},{key:'days',label:'Working days',type:'number'},{key:'status',label:'Status',type:'select',options:['Pending','Approved','Rejected'].map(x=>({value:x,label:x}))},{key:'reason',label:'Reason',type:'textarea'},{key:'appliedAt',label:'Applied on',type:'date'}],
  };
  const submit=async(e:FormEvent)=>{e.preventDefault();setBusy(true);setError('');const output={...values};for(const key of ['departmentId','shiftId','employeeId','days'])if(key in output)output[key]=Number(output[key]);if('checkIn'in output&&!output.checkIn)output.checkIn=null;if('checkOut'in output&&!output.checkOut)output.checkOut=null;try{await onSave(output);}catch(err){setError(err instanceof Error?err.message:'Could not save.');}finally{setBusy(false);}};
  const heading=mode==='create'?`Add ${kindLabel(kind)}`:mode==='edit'?`Edit ${kindLabel(kind)}`:`${kindLabel(kind)} details`;
  return <div className="fixed inset-0 z-50 flex items-end justify-center bg-[#17251f]/35 p-0 backdrop-blur-[2px] sm:items-center sm:p-5" onMouseDown={e=>{if(e.target===e.currentTarget)onClose()}}><section role="dialog" aria-modal="true" aria-label={heading} className="enter max-h-[92dvh] w-full max-w-[560px] overflow-y-auto rounded-t-2xl border border-border bg-card shadow-2xl sm:rounded-2xl">
    <div className="flex items-start justify-between border-b border-border px-6 py-5"><div><p className="mb-1 text-[10px] font-bold uppercase tracking-[.15em] text-primary">{kindLabel(kind)} record</p><h2 className="font-display text-xl font-bold">{heading}</h2><p className="mt-1 text-xs text-muted-foreground">{mode==='view'?'Record information and current details':'Keep the campus directory up to date.'}</p></div><button onClick={onClose} aria-label="Close dialog" className="grid h-8 w-8 place-items-center rounded-lg hover:bg-muted"><X size={16}/></button></div>
    <form onSubmit={submit} className="p-6"><div className="grid gap-4 sm:grid-cols-2">{fields[kind].map(f=><label key={f.key} className={`block ${f.type==='textarea'?'sm:col-span-2':''}`}><span className="mb-1.5 block text-[11px] font-semibold">{f.label}</span>{f.type==='select'?<select required={!readOnly} disabled={readOnly} value={values[f.key]??''} onChange={e=>setValues({...values,[f.key]:e.target.value})} className="h-10 w-full rounded-lg border border-input bg-background px-3 text-xs outline-none focus:ring-2 focus:ring-ring/30"><option value="">Choose {f.label.toLowerCase()}</option>{f.options?.map(o=><option key={o.value} value={o.value}>{o.label}</option>)}</select>:f.type==='textarea'?<textarea disabled={readOnly} required value={values[f.key]??''} rows={3} onChange={e=>setValues({...values,[f.key]:e.target.value})} className="w-full resize-y rounded-lg border border-input bg-background px-3 py-2.5 text-xs outline-none focus:ring-2 focus:ring-ring/30"/>:<input disabled={readOnly} required={f.key!=='checkIn'&&f.key!=='checkOut'} type={f.type||'text'} value={values[f.key]??''} placeholder={(f as any).placeholder||''} onChange={e=>setValues({...values,[f.key]:e.target.value})} className="h-10 w-full rounded-lg border border-input bg-background px-3 text-xs outline-none focus:ring-2 focus:ring-ring/30"/>}</label>)}</div>
      {error&&<p role="alert" className="mt-4 rounded-lg bg-destructive/10 px-3 py-2 text-xs text-destructive">{error}</p>}
      <div className="mt-6 flex justify-end gap-2 border-t border-border pt-4"><button type="button" onClick={onClose} className="h-9 rounded-lg border border-border px-4 text-xs font-semibold hover:bg-muted">{readOnly?'Close':'Cancel'}</button>{!readOnly&&<button disabled={busy} className="h-9 rounded-lg bg-primary px-4 text-xs font-semibold text-primary-foreground disabled:opacity-60">{busy?'Saving…':mode==='create'?'Create record':'Save changes'}</button>}</div>
    </form>
  </section></div>;
}

function StatusBadge({value}:{value:string}){const lower=value.toLowerCase();const cls=lower.includes('present')||lower.includes('approved')||lower==='active'?'bg-[#e4efe8] text-[#35684d]':lower.includes('pending')||lower.includes('late')||lower.includes('leave')?'bg-[#f7ecd9] text-[#94642f]':lower.includes('reject')||lower.includes('absent')||lower.includes('inactive')?'bg-[#f4e5e3] text-[#a04d47]':'bg-muted text-muted-foreground';return <span className={`inline-flex items-center gap-1 rounded-full px-2.5 py-1 text-[9px] font-bold ${cls}`}><i className="h-1.5 w-1.5 rounded-full bg-current opacity-70"/>{value}</span>}
function Avatar({name}:{name:string}){const initials=name.split(' ').map(x=>x[0]).slice(0,2).join('').toUpperCase();const colors=['bg-[#e5ebdf] text-[#4e6e52]','bg-[#f0e4d7] text-[#906640]','bg-[#e5e6ef] text-[#5a6387]','bg-[#f0e0e1] text-[#96565a]'];const index=(name.charCodeAt(0)||0)%colors.length;return <span className={`grid h-8 w-8 shrink-0 place-items-center rounded-full font-display text-[10px] font-bold ${colors[index]}`}>{initials}</span>}
function EmptyState({kind,onAdd}:{kind:Kind;onAdd:()=>void}){return <div className="flex flex-col items-center px-5 py-16 text-center"><div className="mb-4 grid h-14 w-14 place-items-center rounded-2xl bg-secondary text-primary">{kind==='employee'?<Users/>:kind==='department'?<Building2/>:kind==='shift'?<CalendarDays/>:kind==='leave'?<FileClock/>:<Clock3/>}</div><h3 className="font-display text-base font-bold">A little room to grow</h3><p className="mt-1 max-w-sm text-xs leading-5 text-muted-foreground">Nothing matches this view yet. Add a record or adjust your search and filters.</p><button onClick={onAdd} className="mt-5 inline-flex h-9 items-center gap-2 rounded-lg bg-primary px-3.5 text-xs font-semibold text-primary-foreground"><Plus size={14}/> Add {kindLabel(kind)}</button></div>}
function EmptyInline({label}:{label:string}){return <p className="px-6 py-9 text-center text-xs text-muted-foreground">{label}</p>}
function SkeletonPage(){return <div className="animate-pulse space-y-6"><div className="h-9 w-64 rounded-lg bg-muted"/><div className="grid gap-4 sm:grid-cols-2 xl:grid-cols-4">{[1,2,3,4].map(n=><div key={n} className="h-32 rounded-xl bg-muted"/>)}<di **…**

_This response is too long to display in full._
