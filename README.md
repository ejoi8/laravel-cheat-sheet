# laravel-cheat-sheet

Get auth user name/id

        auth()->user()->name;
        auth()->user()->id;

Get auth user role

        auth()->user()->hasRole('admin');

Redirect to rout with notification

        return redirect()->route('route.name')->with('success','notification here');
                        
Get current time

        \Carbon\Carbon::now();
  
Get ip

        \Request::ip();

== Search function ==
   
        public function index(Request $request)
        {
            $data = $this->carian($request)
            ->where('office_id',auth()->user()->office_id)
            ->paginate(10);

                return view('users.index',compact('data'))
                    ->with('i', ($request->input('page', 1) - 1) * 5);
        }

        public function carian($request)
        {
            $data = User::orderBy('id','DESC');

            if ($request->filled('carian')) {
                $data->where('nokp','like','%'.$request->carian);
            }

            if ($request->filled('office_id')) {
                $data->where('office_id',$request->office_id);
            }

            if ($request->filled('department_id')) {
                $data->where('department_id',$request->department_id);
            }

            return $data;
        }
        
Eloquent GroupBy Report
        
        public function index()
        {
                $bil_cawangan = $this->statistik_cawangan()->get()->pluck('penerangan','user_count');
                return view('home',compact('users','bil_cawangan'));
        }
        
        public function statistik_cawangan()
        {
                return DB::table('users')
                         ->select(DB::raw('ref_table.penerangan, count(*) as user_count'))
                         ->join('ref_table', 'users.cawangan_id', '=', 'ref_table.id')
                         ->groupBy('ref_table.penerangan')->orderBy('user_count','DESC');
        }
