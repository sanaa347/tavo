<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Study Planner</title>
  <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/recharts/umd/Recharts.js"></script>
  <style>
    body { font-family: sans-serif; margin: 0; padding: 0; }
    button { cursor: pointer; }
  </style>
</head>
<body>
  <div id="root"></div>
  <script type="text/javascript">
    const { useState, useEffect } = React;
    const { Radar, RadarChart, PolarGrid, PolarAngleAxis, PolarRadiusAxis, Tooltip, ResponsiveContainer } = window.Recharts;

    const classes = ["Korean", "TOPIK", "Math", "Business Communication", "Business Legal", "Accountant", "Management"];
    const categories = ["Assignments", "Homework", "Business", "Exams/Tests prep", "Personal Study"];
    const trackerCategories = ['Productivity','Mood','Sleep'];
    const weeks = Array.from({length:17},(_,i)=>i+1);

    function StudyPlanner() {
      const [currentDate] = useState(new Date());
      const semesterStart = new Date('2025-08-25');

      const getCurrentWeek = ()=>{
        const diff = currentDate - semesterStart;
        const day = 1000*60*60*24;
        const week = Math.floor(diff/day/7)+1;
        return week<1?1:(week>17?17:week);
      }

      const [selectedWeek, setSelectedWeek] = useState(getCurrentWeek());
      const [activeTab, setActiveTab] = useState('dashboard');

      const [tasks, setTasks] = useState(()=>JSON.parse(localStorage.getItem('sp_tasks')||'{}'));
      const [trackers, setTrackers] = useState(()=>JSON.parse(localStorage.getItem('sp_trackers')||'{}'));
      const [cubePlan, setCubePlan] = useState(()=>JSON.parse(localStorage.getItem('sp_cubePlan')||JSON.stringify(weeks.reduce((acc,w)=>({...acc,[`W${w}`]:[]}),{}))));

      useEffect(()=>{ localStorage.setItem('sp_tasks',JSON.stringify(tasks)); },[tasks]);
      useEffect(()=>{ localStorage.setItem('sp_trackers',JSON.stringify(trackers)); },[trackers]);
      useEffect(()=>{ localStorage.setItem('sp_cubePlan',JSON.stringify(cubePlan)); },[cubePlan]);

      const addTask = (title, category, cls)=>{
        const weekKey = `W${selectedWeek}`;
        setTasks(prev=>{
          const weekTasks = prev[weekKey]||[];
          return {...prev,[weekKey]:[...weekTasks,{title,category,class:cls,done:false}]};
        });
      };

      const toggleTaskDone = (index)=>{
        const weekKey = `W${selectedWeek}`;
        setTasks(prev=>{
          const weekTasks = prev[weekKey].map((t,i)=>i===index?{...t,done:!t.done}:t);
          return {...prev,[weekKey]:weekTasks};
        });
      };

      const addCubeEntry = (weekKey)=>{
        const text = prompt(`Add weekly goal/task for ${weekKey}`);
        if(!text) return;
        setCubePlan(prev=>({...prev,[weekKey]:[...(prev[weekKey]||[]),text]}));
      };

      const updateTracker = (category, value)=>{
        const weekKey = `W${selectedWeek}`;
        setTrackers(prev=>({...prev,[weekKey]:{...prev[weekKey],[category]:value}}));
      };

      const calculateRadar = ()=>{
        let classData = {};
        classes.forEach(cls=>{
          let data = {};
          categories.forEach(c=>{
            const weekTasks = tasks[`W${selectedWeek}`]?.filter(t=>t.class===cls && t.category===c) || [];
            data[c] = weekTasks.length?Math.round(weekTasks.filter(t=>t.done).length/weekTasks.length*100):0;
          });
          classData[cls] = data;
        });

        let overall = {};
        categories.forEach(c=>{
          let done=0,total=0;
          Object.values(tasks).forEach(weekArr=>{
            weekArr.forEach(t=>{if(t.category===c){total++;if(t.done) done++;}});
          });
          overall[c] = total?Math.round(done/total*100):0;
        });

        return {classData, overall};
      };

      const {classData, overall} = calculateRadar();

      return React.createElement('div', {className:"flex h-screen"},
        React.createElement('div', {className:"w-48 bg-gray-100 p-4 flex flex-col space-y-2"},
          ['dashboard','tasks','trackers','backup'].map(tab=>
            React.createElement('button', {key: tab, onClick: ()=>setActiveTab(tab), className: `${activeTab===tab?'bg-blue-400 text-white':'bg-white'} p-2 rounded`}, tab.charAt(0).toUpperCase()+tab.slice(1))
          )
        ),
        React.createElement('div', {className:"flex-1 p-6 overflow-auto"},
          activeTab==='dashboard' && React.createElement('div', {className:"space-y-6"},
            React.createElement('h1', {className:"text-2xl font-bold mb-4"}, "Dashboard"),
            React.createElement('div', {className:"grid md:grid-cols-2 gap-6"},
              classes.map(cls=>React.createElement('div',{key:cls,className:"bg-white p-4 rounded-2xl shadow"},
                React.createElement('h2',{className:"font-semibold mb-2"}, cls+" Radar"),
                React.createElement(ResponsiveContainer, {width:"100%",height:250},
                  React.createElement(RadarChart, {data: categories.map(c=>({category:c,value:classData[cls][c]}))},
                    React.createElement(PolarGrid,null),
                    React.createElement(PolarAngleAxis,{dataKey:"category"}),
                    React.createElement(PolarRadiusAxis,{angle:30,domain:[0,100]}),
                    React.createElement(Radar,{dataKey:"value",stroke:"#8884d8",fill:"#8884d8",fillOpacity:0.6}),
                    React.createElement(Tooltip,null)
                  )
                )
              )),
              React.createElement('div',{className:"bg-white p-4 rounded-2xl shadow"},
                React.createElement('h2',{className:"font-semibold mb-2"}, "Overall Radar"),
                React.createElement(ResponsiveContainer, {width:"100%",height:250},
                  React.createElement(RadarChart, {data: categories.map(c=>({category:c,value:overall[c]}))},
                    React.createElement(PolarGrid,null),
                    React.createElement(PolarAngleAxis,{dataKey:"category"}),
                    React.createElement(PolarRadiusAxis,{angle:30,domain:[0,100]}),
                    React.createElement(Radar,{dataKey:"value",stroke:"#82ca9d",fill:"#82ca9d",fillOpacity:0.6}),
                    React.createElement(Tooltip,null)
                  )
                )
              )
            )
          ),
          activeTab==='tasks' && React.createElement('div',{className:"space-y-6"},
            React.createElement('div',{className:"mb-4"},
              React.createElement('label',null,"Week: "),
              React.createElement('select',{value:selectedWeek,onChange:e=>setSelectedWeek(+e.target.value),className:"border p-1 rounded"},
                weeks.map(w=>React.createElement('option',{key:w,value:w},"Week "+w))
              )
            ),
            React.createElement('div',{className:"grid md:grid-cols-2 gap-6"},
              React.createElement('div',{className:"bg-white p-4 rounded-2xl shadow"},
                React.createElement('h2',{className:"font-semibold mb-2"},"Tasks"),
                React.createElement('ul',null,(tasks[`W${selectedWeek}`]||[]).map((t,i)=>
                  React.createElement('li',{key:i,className:"flex justify-between p-2 border-b"},
                    React.createElement('span',null, t.title + " (" + t.category + "/" + t.class + ")"),
                    React.createElement('button',{onClick:()=>toggleTaskDone(i),className:"px-2 py-1 border rounded"}, t.done?'Undo':'Done')
                  )
                )),
                React.createElement('button',{onClick:()=>{const title=prompt('Task title'); const cat=prompt('Category'); const cls=prompt('Class'); if(title&&cat&&cls)addTask(title,cat,cls);},className:"mt-2 px-3 py-1 rounded border"},"Add Task")
              ),
              React.createElement('div',{className:"bg-white p-4 rounded-2xl shadow"},
                React.createElement('h2',{className:"font-semibold mb-2"},"Cube Grid"),
                (cubePlan[`W${selectedWeek}`]||[]).map((entry,i)=>React.createElement('div',{key:i,className:"p-1 border rounded mb-1"}, entry)),
                React.createElement('button',{onClick:()=>addCubeEntry(`W${selectedWeek}`),className:"mt-2 px-3 py-1 rounded border"},"Add Cube Entry")
              )
            )
          ),
          activeTab==='trackers' && React.createElement('div',{className:"space-y-6"},
            React.createElement('h1',{className:"text-2xl font-bold mb-4"},"Trackers"),
            trackerCategories.map(cat=>
              React.createElement('div',{key:cat,className:"bg-white p-4 rounded-2xl shadow mb-4"},
                React.createElement('h2',{className:"font-semibold mb-2"},cat),
                React.createElement('input',{type:"number", min:0, max

