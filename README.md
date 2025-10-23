# https-github.com-AZMATHimport React, { useState, useEffect, useRef } from 'react';
import {
  Terminal,
  Server,
  GitBranch,
  Boxes,
  Activity,
  Scale,
  Package,
  Play,
  CheckCircle2,
  Loader2,
  Copy,
  Settings,
  Github,
  Globe
} from 'lucide-react';
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

// Utility for Tailwind class merging
function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// --- Types ---
interface BuildPhase {
  id: string;
  title: string;
  icon: React.ElementType;
  description: string;
  prompt: string;
  status: 'idle' | 'running' | 'completed' | 'error';
}

// --- Data ---
const INITIAL_PHASES: BuildPhase[] = [
  {
    id: 'phase-1',
    title: 'Phase 1: EKS Cluster Infrastructure',
    icon: Server,
    description: 'Terraform setup for EKS v1.28, Multi-AZ Node Groups, IAM, and Networking in us-east-1.',
    prompt: `Create a Terraform module for AWS EKS cluster v1.28 in us-east-1 with:
- Managed node groups across 3 availability zones
- Auto-scaling group with min=3, max=10 nodes
- IAM roles for cluster and node groups
- Security groups for worker nodes
- EBS CSI driver and Load Balancer controller
- Network policies and VPC configuration
- Output cluster endpoint and kubeconfig`,
    status: 'idle',
  },
  {
    id: 'phase-2',
    title: 'Phase 2: GitOps with ArgoCD',
    icon: GitBranch,
    description: 'Install ArgoCD, configure ApplicationSets, RBAC, and Ingress for declarative CD.',
    prompt: `Create Helm charts and Kubernetes manifests for ArgoCD installation including:
- ArgoCD namespace and CRDs
- ApplicationSet for automatic app discovery
- RBAC configurations
- Ingress setup with SSL
- Sync policies and automated pruning
- Repository credentials setup`,
    status: 'idle',
  },
  {
    id: 'phase-3',
    title: 'Phase 3: Microservices Implementation',
    icon: Boxes,
    description: 'Scaffold 12 sample microservices (Python/Node.js) with Dockerfiles and K8s manifests.',
    prompt: `Create 12 sample microservices (6 Python Flask, 6 Node.js Express) with:
- Dockerfiles for each service
- Kubernetes deployments and services
- ConfigMaps and Secrets management
- Health checks and readiness probes
- Service mesh configuration
- API gateway routing rules`,
    status: 'idle',
  },
  {
    id: 'phase-4',
    title: 'Phase 4: Monitoring & Observability',
    icon: Activity,
    description: 'Deploy Prometheus stack, Grafana dashboards (30+), and CloudWatch logging.',
    prompt: `Deploy Prometheus stack with:
- Prometheus server with 150+ exporters
- Grafana with 30+ pre-configured dashboards
- Alert manager configuration
- Service monitors for all microservices
- Resource usage tracking
- Custom metrics for HPA`,
    status: 'idle',
  },
  {
    id: 'phase-5',
    title: 'Phase 5: Autoscaling Configuration',
    icon: Scale,
    description: 'Implement HPA with custom metrics and VPA for resource optimization.',
    prompt: `Implement HPA and VPA configurations:
- Horizontal Pod Autoscaler with custom metrics
- Vertical Pod Autoscaler for resource optimization
- Cluster autoscaler policies
- Resource requests and limits
- Scaling triggers based on CPU, memory, and custom metrics`,
    status: 'idle',
  },
  {
    id: 'phase-6',
    title: 'Phase 6: Helm Chart Packaging',
    icon: Package,
    description: 'Package all applications and dependencies (MongoDB, Redis, RabbitMQ) with Helm.',
    prompt: `Create Helm charts for:
- MongoDB with persistence and replication
- Redis cluster configuration
- RabbitMQ with queue management
- NGINX Ingress Controller
- All microservices with environment-specific values
- Dependency management and versioning`,
    status: 'idle',
  },
];

export default function EKSBuilderDashboard() {
  // --- State ---
  const [phases, setPhases] = useState<BuildPhase[]>(INITIAL_PHASES);
  const [isBuilding, setIsBuilding] = useState(false);
  const [currentPhaseIndex, setCurrentPhaseIndex] = useState(-1);
  const [logs, setLogs] = useState<string[]>([
    '> Ready to initialize EKS Platform build sequence...',
    '> Awaiting user configuration and start command.',
  ]);
  
  // Configuration State
  const [repoUrl, setRepoUrl] = useState('https://github.com/AZMATH-DOT/eks-microservices-platform');
  const [region, setRegion] = useState('us-east-1');
  const [clusterName, setClusterName] = useState('production-eks-cluster');

  const terminalRef = useRef<HTMLDivElement>(null);

  // --- Effects ---
  // Auto-scroll terminal
  useEffect(() => {
    if (terminalRef.current) {
      terminalRef.current.scrollTop = terminalRef.current.scrollHeight;
    }
  }, [logs]);

  // Build Simulation Loop
  useEffect(() => {
    if (!isBuilding || currentPhaseIndex >= phases.length) {
      if (currentPhaseIndex >= phases.length && isBuilding) {
        setIsBuilding(false);
        addLog('SUCCESS: All phases completed successfully. Repository is ready.');
        addLog(`> Pushed complete codebase to ${repoUrl}`);
      }
      return;
    }

    const phase = phases[currentPhaseIndex];
    
    // Start Phase
    setPhases(prev => prev.map((p, i) => i === currentPhaseIndex ? { ...p, status: 'running' } : p));
    addLog(`\n[${phase.id.toUpperCase()}] Starting: ${phase.title}...`);
    addLog(`> Executing prompt generation...`);

    // Simulate work with varied timing
    const timer = setTimeout(() => {
      // Complete Phase
      setPhases(prev => prev.map((p, i) => i === currentPhaseIndex ? { ...p, status: 'completed' } : p));
      addLog(`[${phase.id.toUpperCase()}] Completed.`);
      
      // Move to next
      setCurrentPhaseIndex(prev => prev + 1);
    }, 3000 + Math.random() * 2000);

    return () => clearTimeout(timer);
  }, [isBuilding, currentPhaseIndex, phases.length]);

  // --- Handlers ---
  const addLog = (msg: string) => {
    const timestamp = new Date().toLocaleTimeString([], { hour12: false });
    setLogs(prev => [...prev, `[${timestamp}] ${msg}`]);
  };

  const handleStartBuild = () => {
    if (isBuilding) return;
    setLogs(['> Initializing automated build sequence...', `> Target Repo: ${repoUrl}`, `> Target Region: ${region}`]);
    setPhases(INITIAL_PHASES);
    setCurrentPhaseIndex(0);
    setIsBuilding(true);
  };

  const copyToClipboard = (text: string, label: string) => {
    navigator.clipboard.writeText(text);
    addLog(`> Copied \"${label}\" to clipboard.`);
  };

  // --- Renderers ---
  const StatusIcon = ({ status }: { status: BuildPhase['status'] }) => {
    switch (status) {
      case 'running': return <Loader2 className="w-5 h-5 text-amber-500 animate-spin" />;
      case 'completed': return <CheckCircle2 className="w-5 h-5 text-emerald-500" />;
      case 'error': return <Activity className="w-5 h-5 text-red-500" />;
      default: return <div className="w-5 h-5 rounded-full border-2 border-slate-700" />;
    }
  };

  return (
    <div className="min-h-screen bg-slate-950 text-slate-300 p-6 font-sans selection:bg-blue-500/30">
      <div className="max-w-7xl mx-auto space-y-6">
        
        {/* Header */}
        <header className="flex flex-col md:flex-row md:items-center justify-between gap-4 pb-6 border-b border-slate-800">
          <div>
            <h1 className="text-3xl font-bold text-white tracking-tight flex items-center gap-3">
              <Boxes className="w-8 h-8 text-blue-500" />
              EKS Platform Builder
            </h1>
            <p className="text-slate-400 mt-2 max-w-2xl">
              Automated generation of Kubernetes-Based Microservices Platform on EKS with Helm, ArgoCD, and Prometheus.
            </p>
          </div>
          <div className="flex items-center gap-3">
            <div className={cn(
              "px-3 py-1 rounded-full text-sm font-medium flex items-center gap-2 border",
              isBuilding 
                ? "bg-amber-500/10 text-amber-400 border-amber-500/20"
                : "bg-emerald-500/10 text-emerald-400 border-emerald-500/20"
            )}>
              <div className={cn("w-2 h-2 rounded-full", isBuilding ? "bg-amber-500 animate-pulse" : "bg-emerald-500")} />
              {isBuilding ? 'Build in Progress' : 'System Ready'}
            </div>
          </div>
        </header>

        <div className="grid grid-cols-1 lg:grid-cols-12 gap-6">
          {/* Left Column: Config & Phases */}
          <div className="lg:col-span-7 space-y-6">
            
            {/* Configuration Card */}
            <section className="bg-slate-900/50 border border-slate-800 rounded-xl p-5 backdrop-blur-sm">
              <div className="flex items-center gap-2 mb-4 text-white font-semibold">
                <Settings className="w-5 h-5 text-blue-400" />
                <h2>Build Configuration</h2>
              </div>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div className="space-y-1.5">
                  <label className="text-xs font-medium text-slate-400 flex items-center gap-1.5">
                    <Github className="w-3.5 h-3.5" /> Target Repository
                  </label>
                  <input 
                    type="text" 
                    value={repoUrl}
                    onChange={(e) => setRepoUrl(e.target.value)}
                    disabled={isBuilding}
                    className="w-full bg-slate-950 border border-slate-800 rounded-md px-3 py-2 text-sm text-white focus:ring-1 focus:ring-blue-500 focus:border-blue-500 transition-colors disabled:opacity-50"
                  />
                </div>
                <div className="space-y-1.5">
                  <label className="text-xs font-medium text-slate-400 flex items-center gap-1.5">
                    <Globe className="w-3.5 h-3.5" /> AWS Region
                  </label>
                  <select 
                    value={region}
                    onChange={(e) => setRegion(e.target.value)}
                    disabled={isBuilding}
                    className="w-full bg-slate-950 border border-slate-800 rounded-md px-3 py-2 text-sm text-white focus:ring-1 focus:ring-blue-500 focus:border-blue-500 transition-colors disabled:opacity-50"
                  >
                    <option value="us-east-1">us-east-1 (N. Virginia)</option>
                    <option value="us-west-2">us-west-2 (Oregon)</option>
                    <option value="eu-central-1">eu-central-1 (Frankfurt)</option>
                  </select>
                </div>
                <div className="space-y-1.5 md:col-span-2">
                  <label className="text-xs font-medium text-slate-400 flex items-center gap-1.5">
                    <Server className="w-3.5 h-3.5" /> Cluster Name
                  </label>
                  <input 
                    type="text" 
                    value={clusterName}
                    onChange={(e) => setClusterName(e.target.value)}
                    disabled={isBuilding}
                    className="w-full bg-slate-950 border border-slate-800 rounded-md px-3 py-2 text-sm text-white focus:ring-1 focus:ring-blue-500 focus:border-blue-500 transition-colors disabled:opacity-50"
                  />
                </div>
              </div>
            </section>

            {/* Phases List */}
            <section className="space-y-3">
              <div className="flex items-center justify-between">
                <h2 className="text-lg font-semibold text-white">Implementation Phases</h2>
                <span className="text-xs text-slate-500">{phases.filter(p => p.status === 'completed').length} / {phases.length} completed</span>
              </div>
              
              <div className="space-y-3">
                {phases.map((phase, index) => (
                  <div 
                    key={phase.id}
                    className={cn(
                      "group relative overflow-hidden rounded-lg border p-4 transition-all duration-300",
                      phase.status === 'running' ? "bg-slate-900 border-blue-500/50 shadow-[0_0_15px_rgba(59,130,246,0.1)]" : 
                      phase.status === 'completed' ? "bg-slate-900/50 border-emerald-900/50" :
                      "bg-slate-900/30 border-slate-800"
                    )}
                  >
                    {/* Progress Bar Background for running state */}
                    {phase.status === 'running' && (
                      <div className="absolute bottom-0 left-0 h-0.5 bg-blue-500 animate-[loading_3s_ease-in-out_infinite] w-full" />
                    )}

                    <div className="flex items-start gap-4">
                      <div className={cn(
                        "mt-1 p-2 rounded-md",
                        phase.status === 'running' ? "bg-blue-500/20 text-blue-400" :
                        phase.status === 'completed' ? "bg-emerald-500/10 text-emerald-400" :
                        "bg-slate-800 text-slate-400"
                      )}>
                        <phase.icon className="w-5 h-5" />
                      </div>
                      
                      <div className="flex-1 min-w-0">
                        <div className="flex items-center justify-between mb-1">
                          <h3 className={cn(
                            "font-medium truncate",
                            phase.status === 'running' ? "text-blue-400" :
                            phase.status === 'completed' ? "text-emerald-400" :
                            "text-slate-200"
                          )}>
                            {phase.title}
                          </h3>
                          <StatusIcon status={phase.status} />
                        </div>
                        <p className="text-sm text-slate-400 line-clamp-2">
                          {phase.description}
                        </p>
                        
                        <div className="mt-3 flex items-center gap-2">
                          <button
                            onClick={() => copyToClipboard(phase.prompt, `${phase.id} prompt`)}
                            className="text-xs flex items-center gap-1.5 px-2 py-1 rounded-md bg-slate-950 hover:bg-slate-800 text-slate-400 hover:text-white transition-colors border border-slate-800"
                          >
                            <Copy className="w-3 h-3" /> Copy Prompt
                          </button>
                        </div>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            </section>
          </div>

          {/* Right Column: Actions & Terminal */}
          <div className="lg:col-span-5 space-y-6 sticky top-6 h-fit">
            {/* Action Card */}
            <div className="bg-slate-900/50 border border-slate-800 rounded-xl p-6 backdrop-blur-sm">
              <h3 className="text-lg font-semibold text-white mb-2">Execute Automation</h3>
              <p className="text-sm text-slate-400 mb-6">
                This will generate all necessary Terraform, Kubernetes, and Helm code, then push it to your designated repository.
              </p>
              
              <button
                onClick={handleStartBuild}
                disabled={isBuilding}
                className={cn(
                  "w-full flex items-center justify-center gap-2 py-3 px-4 rounded-lg font-semibold text-white transition-all duration-200",
                  isBuilding 
                    ? "bg-slate-700 cursor-not-allowed opacity-75"
                    : "bg-blue-600 hover:bg-blue-500 hover:shadow-lg hover:shadow-blue-500/20 active:scale-[0.98]"
                )}
              >
                {isBuilding ? (
                  <>
                    <Loader2 className="w-5 h-5 animate-spin" />
                    Building Platform...
                  </>
                ) : (
                  <>
                    <Play className="w-5 h-5 fill-current" />
                    Start Automated Build
                  </>
                )}
              </button>
            </div>

            {/* Terminal Output */}
            <div className="bg-[#0d1117] border border-slate-800 rounded-xl overflow-hidden flex flex-col h-[500px]">
              <div className="bg-slate-900 border-b border-slate-800 px-4 py-2 flex items-center justify-between shrink-0">
                <div className="flex items-center gap-2">
                  <Terminal className="w-4 h-4 text-slate-400" />
                  <span className="text-xs font-medium text-slate-300">Build Logs</span>
                </div>
                <div className="flex gap-1.5">
                  <div className="w-2.5 h-2.5 rounded-full bg-red-500/20 border border-red-500/50" />
                  <div className="w-2.5 h-2.5 rounded-full bg-amber-500/20 border border-amber-500/50" />
                  <div className="w-2.5 h-2.5 rounded-full bg-emerald-500/20 border border-emerald-500/50" />
                </div>
              </div>
              
              <div 
                ref={terminalRef}
                className="flex-1 p-4 overflow-y-auto font-mono text-xs leading-relaxed scrollbar-thin scrollbar-thumb-slate-700 scrollbar-track-transparent"
              >
                {logs.map((log, i) => (
                  <div key={i} className={cn(
                    "mb-1 break-words",
                    log.includes('ERROR') ? 'text-red-400' :
                    log.includes('SUCCESS') || log.includes('Completed') ? 'text-emerald-400' :
                    log.includes('Starting') ? 'text-blue-400' :
                    log.startsWith('>') ? 'text-slate-500' :
                    'text-slate-300'
                  )}>
                    {log}
                  </div>
                ))}
                {isBuilding && (
                  <div className="flex items-center gap-1 text-slate-500 mt-2">
                    <span className="animate-pulse">▋</span>
                  </div>
                )}
              </div>
            </div>

          </div>
        </div>
      </div>
    </div>
  );
}
-DOT-eks-microservices-platform1
