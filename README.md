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

        use Illuminate\Support\Facades\DB;
        
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
        
Eloquent find user DoesntHave record in other table and with status count
                
                $users = User::where('cawangan_id',auth()->user()->cawangan_id)
                    ->whereDoesntHave('table_x', function(Builder $query) {
                        $query->where('status_id',1)
                        ->havingRaw('COUNT(status_id) > 0');
                        

Abort if user doesnt meet the condition. Ex: block user from access others users information
                
                public function show(Staff $Staff)
                {
                        ...
                        abort_if(auth()->user()->sekat($kad_owner,$cawangan_id_kad_owner),403);
                        ...
                }
                
                //in Model
                public function sekat($owner,$cawangan_id)
                {
                        if ($this->hasRole('Admin')) {
                             return $this->cawangan_id != $cawangan_id;
                        }

                        if ($this->hasRole('User')) {
                             return $this->id != $owner;
                        }
                }
                
                

LARAVEL BLADE SYNTAX

                <strong>Name:</strong>
                {!! Form::text('field_name', null, array('placeholder' => 'field_name','class' => 'form-control' . ($errors->has('field_name') ? ' is-invalid' : null) )) !!}
                @if ($errors->has('field_name'))
                <div class="invalid-feedback">
                    <strong>{{ $errors->first('field_name') }}</strong>
                </div>
                @endif

                <strong>Country:</strong>
                {!! Form::select('country_list', $ref_table->pluck('nama','id'),null,['class' => 'form-control'. ($errors->has('country_list') ? ' is-invalid' : null) ,'placeholder' => 'Choose country...']); !!}
                @if ($errors->has('country_list'))
                <div class="invalid-feedback">
                    <strong>{{ $errors->first('country_list') }}</strong>
                </div>
                @endif


                <strong>Password:</strong>
                {!! Form::password('password', array('placeholder' => 'Password','class' => 'form-control' . ($errors->has('password') ? ' is-invalid' : null) )) !!}
                @if ($errors->has('password'))
                <div class="invalid-feedback">
                    <strong>{{ $errors->first('password') }}</strong>
                </div>
                @endif
                <strong>Confirm Password:</strong>
                {!! Form::password('password_confirmation', array('placeholder' => 'Confirm Password','class' => 'form-control'. ($errors->has('password_confirmation') ? ' is-invalid' : null) )) !!}
                @if ($errors->has('password_confirmation'))
                <div class="invalid-feedback">
                    <strong>{{ $errors->first('password_confirmation') }}</strong>
                </div>
                @endif


                <strong>Tarikh lahir:</strong>
                {!! Form::text('tarikh_lahir', null, array('placeholder' => 'Tarikh lahir','class' => 'form-control datemask'. ($errors->has('roles') ? ' is-invalid' : null) )) !!}
                @if ($errors->has('tarikh_lahir'))
                <div class="invalid-feedback">
                    <strong>{{ $errors->first('tarikh_lahir') }}</strong>
                </div>
                @endif

                <strong>Alamat:</strong>
                {!! Form::textarea('alamat', null, array('placeholder' => 'Alamat','class' => 'form-control'. ($errors->has('alamat') ? ' is-invalid' : null) )) !!}
                @if ($errors->has('alamat'))
                <div class="invalid-feedback">
                    <strong>{{ $errors->first('alamat') }}</strong>
                </div>
                @endif


                <script>
                    $(document).ready(function() {
                        //need to include related script such as jquery,select,inputmask,bootstrapDualListbox

                        //select2
                        $('.selectComponent').select2();
                        //Datemask dd/mm/yyyy
                        $('.datemask').inputmask('yyyy-mm-dd', { 'placeholder': 'yyyy-mm-dd' })
                        //adminlte 3 default collapse card 
                        $('#login-akaun').CardWidget('collapse');
                        //adminlte 3 default collapse card 
                        $('#maklumat-tambahan').CardWidget('collapse');
                        //alternative for select2. More convinient to select record
                        $("#user_id").bootstrapDualListbox();

                        //show filename after select file
                            $('input[type="file"]').change(function(e){
                                    var fileName = e.target.files[0].name;
                                    alert('The file "' + fileName +  '" has been selected.');
                                    var $this = $(this);
                                    $this.next().html($this.val().split('\\').pop());
                                });

                    });
                </script>

Create Laravel logs Traits (create inside app/IzzulTraits/helper.php)

                <?php 

                namespace App\IzzulTraits;

                trait helper
                {
                        public function logfile($array_data)
                        {

                                //DATE
                                $TarikhLog = date("d-m-Y [h:i:s A]");
                                $day = substr($TarikhLog, 0, 2);
                                $month = substr($TarikhLog, 3,2);
                                $year = substr($TarikhLog, 6, 4);
                                //IP
                                $ip = $_SERVER['REMOTE_ADDR'];
                                //Text Log File
                                $myFile = "$year-$month.log";
                                $fh = fopen($myFile, 'a') or die("can't open file");
                                // $stringData = " \n";
                                $stringData = "$year-$month-$day|$TarikhLog|$ip";
                                foreach ($array_data as $data) {
                                        $stringData = $stringData."|" .$data; 
                                }
                                $stringData = $stringData."\n";

                                fwrite($fh, $stringData);
                                fclose($fh);
                        }
                }
                
                // add in controller: use App\IzzulTraits\helper; 
                // add in the controller class: use helper; 
                // dont forget to: composer dump autoload
                // call inside controller function: $this->logfile([$request->email,'login']);
