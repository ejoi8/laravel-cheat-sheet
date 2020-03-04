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
   
   $data = $this->carian($request)
            ->where('cawangan_id',auth()->user()->cawangan_id)
            ->paginate(10);
   
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
